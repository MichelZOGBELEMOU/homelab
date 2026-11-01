# Mail Requirements

## Scope

Provide internal mail between homelab accounts before considering notification integrations or public mail.

Initial users demonstrated by the supplied test:

- Alice
- Michel

Public SMTP, webmail, spam filtering, DKIM signing, and external deliverability are deferred.

## Functional Requirements

- Accept mail for `mail.home.lab`.
- Deliver valid local recipients into their Maildir mailboxes.
- Allow users to read mail using IMAP over TLS.
- Resolve the mail server through internal DNS.
- Retain logs sufficient to trace submission, queueing, and delivery.
- Reject unauthorized relay to non-local domains.

## Network Requirements

Required or conditional paths, not confirmed active listeners:

- TCP 25: internal SMTP reception where remote senders require it.
- TCP 993: IMAP with implicit TLS, if enabled.
- TCP 143: IMAP with STARTTLS, if enabled; require TLS before authentication.
- TCP 587: authenticated client submission, only if configured.
- TCP/UDP 53: access to approved internal DNS resolvers.

Do not open every listed port automatically. Confirm listeners and approve only the paths actually used.

Firewall requirements:

- Limit client access to approved ADMIN/CLIENT hosts or subnets.
- Restrict destination to the confirmed mail-server address.
- Keep Guest/IoT and DMZ access denied unless explicitly approved.
- Do not forward public WAN ports to the mail service.


## Backup and Recovery Requirements

Back up:

- `/etc/postfix/`
- `/etc/dovecot/`
- User Maildir directories, preserving ownership and permissions.
- Required account/authentication data through a protected backup process.
- TLS material, with encrypted storage and restricted access.

Recovery validation must demonstrate:

1. Restore configuration and required accounts.
2. Restore a test mailbox with correct permissions.
3. Start Postfix and Dovecot successfully.
4. Read a restored message over TLS.
5. Send and receive a new internal message.

Backup destination, schedule, retention, RPO, RTO, and restore-test result: Not yet documented.

## Monitoring Requirements

Define checks and responses for:

- Postfix or Dovecot unavailable: inspect service state and recent logs.
- Queue growth or oldest-message age: inspect deferred-delivery reasons.
- Mail storage capacity/inodes: investigate growth before delivery fails.
- TLS expiry: renew before expiration and retest client access.
- Repeated authentication failures: investigate source and account activity.
- Synthetic delivery/retrieval failure: trace DNS, SMTP, storage, and IMAP.

Monitoring implementation and alert thresholds: Deferred.

