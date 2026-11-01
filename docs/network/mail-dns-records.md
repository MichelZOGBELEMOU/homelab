# Mail DNS Records

## Purpose

Document internal DNS records and validation evidence for the homelab mail service.

## Service Names

- Mail domain: `mail.home.lab`
- Server hostname: `mail.servers.home.lab`
- Server address: `10.10.20.100`
- Client resolvers: `10.10.20.12`, `10.10.20.13`
- Mail address format: `user@mail.home.lab`

## Configured Forward Records

In the `home.lab` zone:

    mail IN A  10.10.20.100
    mail IN MX 10 mail.servers.home.lab.

In the `servers.home.lab` zone:

    mail IN A 10.10.20.100

Resulting records:

    mail.home.lab.         IN A  10.10.20.100
    mail.home.lab.         IN MX 10 mail.servers.home.lab.
    mail.servers.home.lab. IN A  10.10.20.100

The MX record directs mail for `@mail.home.lab` to `mail.servers.home.lab`.

The previous `mail.home.lab` CNAME must not coexist with the A and MX records. The MX target must resolve directly rather than through a CNAME.

## Reverse DNS

Reverse DNS is reported as configured.

Expected record in the `20.10.10.in-addr.arpa` zone:

    100 IN PTR mail.servers.home.lab.

Expected lookup:

    10.10.20.100 → mail.servers.home.lab.

Status: Configured according to the operator; lookup output still needs to be recorded.

## Validation Evidence

### MX Lookup — Passed Through First Resolver

Validation time: 2026-11-01 23:39:49 KST.

Command:

    dig @10.10.20.12 mail.home.lab MX

Observed answer:

    mail.home.lab. 86400 IN MX 10 mail.servers.home.lab.

Result:

- Resolver `10.10.20.12` returns the expected MX record.
- The MX preference is `10`.
- The target is `mail.servers.home.lab`.
- The earlier CNAME-only response is no longer present in this lookup.

This is a recursive resolver response, not direct authoritative-server evidence. It validates DNS discovery, not SMTP delivery.

### Remaining Client-Side Checks

    dig @10.10.20.12 mail.home.lab A +noall +answer
    dig @10.10.20.13 mail.home.lab A +noall +answer
    dig @10.10.20.13 mail.home.lab MX +noall +answer

    dig @10.10.20.12 mail.servers.home.lab A +noall +answer
    dig @10.10.20.13 mail.servers.home.lab A +noall +answer

    dig @10.10.20.12 -x 10.10.20.100 +noall +answer
    dig @10.10.20.13 -x 10.10.20.100 +noall +answer

Expected:

- Both forward names resolve to `10.10.20.100`.
- Both resolvers return the same MX record.
- Reverse lookup returns `mail.servers.home.lab.`

### Authoritative Consistency

Query the actual primary and secondary authoritative server addresses directly.

Verify:

- Both serve the updated `home.lab` zone serial.
- Both return the configured A and MX records.
- No CNAME remains at `mail.home.lab`.
- The relevant forward and reverse zones are consistent.

Authoritative server addresses and direct query evidence: Needs verification.

## DNS Change Procedure

1. Back up the affected zone configuration.
2. Update records in the correct zone.
3. Increment the zone serial.
4. Validate with `named-checkzone`.
5. Reload the zone.
6. Confirm secondary replication.
7. Test through both client resolvers.
8. Record the result in this document.

If authoritative responses are correct but a resolver returns old data, allow the cache to expire or clear the relevant cache on that resolver.

## Address Ownership

Existing `docs/services/reverse-proxy-service.md` also identifies `10.10.20.100:8080` as the File Browser backend.

Confirm whether mail and File Browser intentionally share one host or whether the documentation is outdated. Separate machines must not use the same address.

## Public DNS

This phase uses internal DNS only.

Public MX, SPF, DKIM, DMARC, and provider-controlled reverse DNS are deferred. Internal PTR configuration does not establish public reverse DNS.

## Validation Status

- [x] Forward A and MX configuration supplied.
- [x] Reverse DNS reported as configured.
- [x] Updated MX response verified through `10.10.20.12`.
- [ ] Updated A responses recorded.
- [ ] MX resolution verified through `10.10.20.13`.
- [ ] PTR lookup output recorded.
- [ ] Authoritative primary/secondary consistency verified.
- [ ] Address ownership clarified.
