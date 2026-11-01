# Mail Security Notes

## Scope

The mail service is intended for internal homelab use.

- Postfix: SMTP reception and local delivery.
- Dovecot: IMAP and authentication.
- Maildir: mailbox storage.
- Mail server: `mail.servers.home.lab`
- Client connection name: `mail.home.lab`
- Observed mail-server address: `10.10.20.100`

Public SMTP exposure is outside the current scope. WAN forwarding and external reachability have not yet been verified.

## Authentication and TLS

Supplied Dovecot settings:

    ssl = required
    ssl_server_cert_file = /etc/dovecot/ssl/mail.crt
    ssl_server_key_file = /etc/dovecot/ssl/mail.key

    auth_mechanisms = plain login
    auth_allow_cleartext = no

Intended policy:

- Require TLS before password authentication.
- Reject password authentication over unsecured connections.

PLAIN and LOGIN do not encrypt passwords themselves; they depend on TLS.

Validate the effective configuration:

    dovecot --version
    sudo doveconf -n

Authentication backend, username format, and enforcement tests: Needs verification.

## Certificate Requirements

- Certificate: `/etc/dovecot/ssl/mail.crt`
- Private key: `/etc/dovecot/ssl/mail.key`

Requirements:

- Certificate SAN must cover `mail.home.lab` when clients use that name.
- Include `mail.servers.home.lab` if clients also connect using that hostname.
- Clients must trust the certificate issuer or explicitly trust the self-signed certificate.
- Restrict private-key access to the required service context.
- Monitor expiry and document renewal.

Inspect the certificate and key permissions:

    sudo openssl x509 -in /etc/dovecot/ssl/mail.crt -noout -subject -issuer -dates -ext subjectAltName
    sudo stat -c '%a %U:%G %n' /etc/dovecot/ssl/mail.key

Test whichever IMAP listener is enabled:

    openssl s_client -connect mail.home.lab:993 -servername mail.home.lab -verify_hostname mail.home.lab -verify_return_error

    openssl s_client -starttls imap -connect mail.home.lab:143 -servername mail.home.lab -verify_hostname mail.home.lab -verify_return_error

For a private CA, add `-CAfile /path/to/lab-ca.crt`.

Do not disable certificate verification as the final acceptance test. Follow TLS validation with an actual IMAP login and message-retrieval test.

## Postfix Authentication Boundary

Supplied Dovecot socket configuration:

    service auth {
      unix_listener /var/spool/postfix/private/auth {
        mode = 0660
        user = postfix
        group = postfix
      }
    }

This provides an authentication socket for Postfix. It does not prove SMTP AUTH or authenticated submission is enabled.

Verify separately:

- Postfix SASL configuration.
- Submission listener and TLS policy.
- Authentication backend.
- Rejection of authentication without TLS.

Dovecot TLS settings do not configure Postfix SMTP TLS.

## Observed Firewall Rules

### ADMIN to SRV

Ruleset: `ADMIN-TO-SRV`

Observed mail-specific rules:

- Rule 130: accept TCP destination port 25; 0 packets.
- Rule 140: accept TCP destination port 143; 0 packets.
- Rule 150: accept TCP destination port 993; 9 packets.
- Rule 160: accept TCP destination port 587; 6 packets.

Connection-state rules:

- Rule 10: accept established.
- Rule 20: accept related.
- Rule 30: drop invalid.
- Default action: drop.

These mail rules permit matching ADMIN-to-SRV traffic. The displayed conditions contain no destination-address restriction, so they are not limited to the mail server.

Counters show traffic matched the 993 and 587 rules. They do not prove successful TLS negotiation, authentication, message submission, or retrieval.

### SRV to ADMIN

Ruleset: `SRV-TO-ADMIN`

Observed mail-specific rules:

- Rule 50: accept TCP destination port 25; 8 packets.
- Rule 60: accept TCP destination port 143; 0 packets.
- Rule 70: accept TCP destination port 993; 0 packets.
- Rule 80: accept TCP destination port 587; 0 packets.

Connection-state rules:

- Rule 10: accept established.
- Rule 20: accept related.
- Rule 30: drop invalid.
- Default action: drop.These mail rules allow matching connections from SRV toward mail-service ports in ADMIN. They are not required merely to return responses to an ADMIN client connecting to the mail server.

For ADMIN-initiated sessions, replies normally use the established-state rule. Related traffic is also allowed by the existing policy.

The 8 packets on rule 50 indicate matching SRV-to-ADMIN SMTP traffic, not proof of successful mail delivery. Identify its source, destination, and purpose before removing the rule.

## Firewall Hardening Requirements

### Restrict ADMIN-to-SRV Destinations

After confirming mail-server address ownership:

- Restrict mail rules to the mail-server IP or a dedicated address group.
- Restrict source clients where practical.
- Enable only ports required by the deployed service.

Port purposes:

- TCP 25: SMTP reception, where required.
- TCP 587: authenticated submission, if configured.
- TCP 993: IMAP with implicit TLS.
- TCP 143: IMAP with STARTTLS, if required.

Opening both IMAP ports is not mandatory. Keep only the access methods actually supported and tested.

### Review Reverse-Direction Rules

For normal ADMIN mail-client access:

- Keep established/related return handling.
- Remove SRV-to-ADMIN mail-port allowances if no server-initiated requirement exists.

If an ADMIN host intentionally receives SMTP from SRV, document that requirement and restrict the rule to the specific source and destination.

Do not remove the matching SMTP rule solely because reverse-direction rules are usually unnecessary; investigate its observed traffic first.

### Verify Rule Attachment and Exposure

The supplied output shows named rulesets and counters. Confirm their zone/interface attachment to establish the actual traffic paths.

Also verify:

- Guest/IoT and DMZ cannot access mail unless explicitly approved.
- No public WAN forwarding exposes the service.
- Other rules do not grant broader access.

## Firewall Validation

On the mail server:

    sudo ss -lntp

On VyOS:

    show firewall ipv4 name ADMIN-TO-SRV
    show firewall ipv4 name SRV-TO-ADMIN

From an approved ADMIN client:

- Test the configured TLS-protected IMAP endpoint.
- Test authenticated submission if enabled.
- Send and retrieve a test message.
- Compare firewall counters before and after testing.
- Correlate client results with Postfix and Dovecot logs.

From an unapproved source:

- Verify prohibited connections fail.
- Confirm the intended deny policy handles the test.

After tightening rules, verify new connections as well as existing sessions. Established sessions may remain allowed after a service-specific rule is removed.

## Relay Restrictions

Firewall access does not prevent an open relay by itself.

Inspect:

    sudo postconf mynetworks smtpd_relay_restrictions smtpd_recipient_restrictions
    sudo postconf -n
    sudo postconf -M

Requirements:

- Accept valid local recipients.
- Reject unauthorized relay to non-local domains.
- Limit trusted networks to approved sources.
- Require TLS and authentication for submission if enabled.

Test relay rejection from an untrusted source. Stop after the RCPT response; do not submit unsolicited internet mail.

## DNS Evidence

On 2026-11-01 at 23:39:49 KST, resolver `10.10.20.12` returned:

    mail.home.lab. IN MX 10 mail.servers.home.lab.

This validates MX resolution through the first resolver only.

Remaining evidence:

- Updated A-record responses.
- MX lookup through `10.10.20.13`.
- PTR lookup.
- Authoritative primary/secondary consistency.

## Mailbox and Backup Protection

- Restrict Maildir access to its owner and required service processes.
- Confirm Postfix and Dovecot use the same mailbox location.
- Protect configuration, authentication data, and TLS private keys.
- Encrypt backups containing mail or credentials.
- Preserve ownership and permissions during restoration.
- Test recovery using a restored mailbox and a new delivery.

Never commit passwords, password hashes, private keys, or sensitive mailbox contents to GitHub.Backup schedule, retention, and restore evidence: Not yet documented.

## Logging and Monitoring

Monitor:

- Postfix and Dovecot availability.
- Delivery failures and deferred messages.
- Queue growth and oldest-message age.
- Authentication failures.
- TLS errors and certificate expiry.
- Mail-storage capacity and inode usage.

Each alert must have an investigation or recovery action. Sanitize logs before publishing portfolio evidence.

## Change and Rollback

1. Save protected copies of service and firewall configuration.
2. Record the reason, affected traffic, and expected result.
3. Investigate SRV-to-ADMIN rule 50 traffic before changing it.
4. Apply narrowly scoped changes in small batches.
5. Use a confirmed commit or equivalent rollback protection for VyOS changes.
6. Retest new connections, delivery, TLS, and retrieval.
7. Restore the previous rules if required traffic fails.
8. Save the validated configuration and update documentation.

Keep mailbox data intact during configuration rollback.

## Security Validation Status

- [x] Dovecot authentication/TLS settings recorded.
- [x] MX resolution verified through the first resolver.
- [x] Both directional mail firewall rulesets reviewed.
- [x] Traffic counters observed for ADMIN-to-SRV ports 993/587 and SRV-to-ADMIN port 25.
- [ ] Mail-server IP ownership confirmed.
- [ ] Mail firewall rules restricted to approved endpoints.
- [ ] SRV-to-ADMIN SMTP traffic purpose identified.
- [ ] Unnecessary reverse-direction mail rules removed.
- [ ] Ruleset attachment and WAN exposure verified.
- [ ] Certificate hostname, trust, and key permissions validated.
- [ ] IMAP login and retrieval over TLS demonstrated.
- [ ] Password authentication without TLS rejected.
- [ ] Authenticated submission validated, if enabled.
- [ ] Unauthorized relay rejected.
- [ ] Unauthorized network access rejected.
- [ ] Remaining DNS checks completed.
- [ ] Backup and restore test recorded.
