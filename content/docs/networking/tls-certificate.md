---
title: "TLS Certificate Management"
description: "Verify certificate issuance, chains, renewal automation, and TLS configuration before they expire on you."
icon: "lock"
weight: 730
toc: true
tags: ["tls", "certificates", "security", "networking"]
---

Almost every certificate outage has the same shape: something was issued once by hand, the person who did it moved on, and the calendar reminder went to a mailbox nobody reads. The rest are caused by an incomplete chain that works in a browser and fails in every non-browser client. This checklist covers issuance and key handling, chain and SAN correctness, automated renewal and the monitoring that proves it is working, the TLS configuration itself, and what to do when a certificate expires anyway.

{{< alert context="info" text="**Who runs this:** the team owning the endpoint, with the platform or security team reviewing key handling and renewal automation. **When:** before a new endpoint goes live, and quarterly across the estate." />}}

## 1. Inventory and ownership {#inventory-and-ownership}

- [ ] **Every certificate in the estate is in a single inventory** — including internal services, load balancers, mail servers, VPN concentrators, and the ones on appliances that nobody thinks of as servers.
- [ ] **Each certificate has a named owning team** — an unowned certificate is one that expires, since nobody feels responsible for a renewal that has not failed yet.
- [ ] **The inventory records issuer, expiry, SANs, key type, and renewal method** — renewal method being the field that distinguishes automated from a person with a reminder.
- [ ] **Certificate Transparency logs are monitored for your domains** — this both catches mis-issuance and finds certificates that teams obtained without telling anyone.
- [ ] **Internal CA-issued certificates are inventoried alongside public ones** — internal certificates expire just as hard, and are usually the ones without automation.
- [ ] **Client certificates and their expiry are tracked separately** — a mutual TLS client certificate expiring breaks the caller, so the alert must reach the caller's team rather than the server's.

## 2. Issuance and validation {#issuance-and-validation}

- [ ] **Every certificate is issued through the documented process, not ad hoc** — a certificate obtained on a laptop with a personal ACME account is unrenewable by anyone else.
- [ ] **The validation method is appropriate to the environment** — DNS-01 for wildcards and for hosts not publicly reachable, HTTP-01 only where port 80 will remain reachable for the life of the automation.
- [ ] **The certificate authority is chosen deliberately and recorded** — including whether the environment pins a CA or requires a particular trust store.
- [ ] **CAA records restrict which CAs may issue for your domains** — this is a cheap, DNS-level control that blocks issuance by an attacker who compromises a different validation path.
- [ ] **Certificate lifetimes are short and renewal is expected to be routine** — treat a certificate as something replaced every few weeks, because industry maximum lifetimes continue to shorten and long-lived certificates hide broken automation.
- [ ] **Wildcard certificates are used only where justified** — one wildcard key compromise exposes every host in the domain, and the blast radius of a rotation is equally wide.
- [ ] **Staging or test endpoints of the CA are used while building automation** — production rate limits are low, and burning them mid-outage is a self-inflicted second incident.

## 3. Key handling {#key-handling}

- [ ] **Private keys are generated on the system that will use them, or in an HSM** — a key emailed as an archive has been compromised regardless of what happened next.
- [ ] **Key algorithm and size meet current guidance** — RSA 2048 bit minimum, or ECDSA P-256, with ECDSA preferred for its smaller handshake cost.
- [ ] **Private key files are readable only by the service account that needs them** — mode 0600 or tighter, and never in a world-readable configuration directory.
- [ ] **No private key or PFX bundle is committed to a repository** — scan history, not just the working tree, since a deleted key is still in the git objects.
- [ ] **Keys are rotated at renewal rather than reused** — reusing a key across renewals means a quiet key compromise persists across every subsequent certificate.
- [ ] **A key compromise procedure exists and names who can revoke** — revocation, reissue with a fresh key, and redeployment, with the steps written down before you need them at speed.
- [ ] **Keys used by more than one host are distributed through the secret manager** — not copied by hand between servers, which is how they end up in a backup archive or a chat message.

## 4. Chain and SAN correctness {#chain-and-san-correctness}

- [ ] **The server sends the full chain up to but not including the root** — omitting the intermediate is the classic failure that works in browsers, which cache intermediates, and fails for API clients and mobile apps, which do not.
- [ ] **The chain is verified from a clean machine with no prior cache** — test with a tool that reports the chain explicitly rather than a browser you have already visited the site from.
- [ ] **No expired or superseded cross-signed intermediate is present in the bundle** — a stale extra intermediate in the chain causes validation failures on strict clients, most memorably with legacy OpenSSL versions.
- [ ] **Every hostname clients actually use is in the SAN list** — apex and www, every alias, and any internal hostname used by health checks or service-to-service calls.
- [ ] **The certificate is not relied on for the deprecated Common Name field** — modern clients ignore CN entirely and validate against SAN only.
- [ ] **Certificates behind a load balancer match the hostname the backend is addressed by** — re-encryption to a backend addressed by IP or internal DNS name fails validation unless that name is in the SAN.

```bash
# verify the served chain from outside, with no local cache in play
openssl s_client -connect example.com:443 -servername example.com \
  -showcerts </dev/null 2>/dev/null | openssl x509 -noout -dates -ext subjectAltName
```

## 5. Automated renewal {#automated-renewal}

- [ ] **Renewal is automated end to end, including reloading the service** — a renewed certificate sitting on disk while the process still holds the old one in memory is an outage with a correct file on disk.
- [ ] **Renewal is attempted well before expiry, typically at one third of remaining lifetime** — this leaves many days of retries before anyone needs to be woken up.
- [ ] **The renewal job's failures are alerted on, not just logged** — a cron job that has silently failed for six weeks is the single most common root cause of certificate expiry.
- [ ] **The full renewal path has been exercised at least once deliberately** — force a renewal in a maintenance window rather than discovering at 3am that the ACME account key was lost.
- [ ] **ACME account keys and DNS API credentials are backed up and access is not tied to one person** — losing the account key means re-registering, and losing the DNS credential means DNS-01 renewals stop.
- [ ] **Rate limits and the retry policy are understood** — a tight retry loop against a CA gets you rate limited precisely when you need issuance most.
- [ ] **Certificates distributed to multiple nodes propagate automatically** — the renewal must update every node and every terminating device, not just the one where the client runs.
- [ ] **Renewal works when the service is degraded** — HTTP-01 validation through the same load balancer that is currently failing cannot renew you out of an incident.

{{< alert context="warning" text="**Blocking:** an endpoint whose renewal automation has never completed a real renewal, end to end including service reload, is not ready for production. An untested renewal path is an expiry scheduled for a date you have not chosen." />}}

## 6. Expiry monitoring {#expiry-monitoring}

- [ ] **Expiry is monitored by connecting to the live endpoint, not by reading a file** — the only thing that matters is the certificate the server actually presents to clients.
- [ ] **Alerts fire at multiple thresholds** — a ticket at 30 days, a warning at 14, and a page at 7, so a missed ticket still has two more chances to be caught.
- [ ] **Alerts go to a team rotation, not an individual mailbox** — the classic failure is a reminder addressed to someone who left the company.
- [ ] **Monitoring covers every SNI hostname on shared endpoints** — a load balancer serving twenty certificates is twenty separate expiry risks, and checking the default one tells you nothing about the other nineteen.
- [ ] **Non-HTTPS TLS endpoints are monitored too** — SMTP with STARTTLS, LDAPS, database TLS, message brokers, and internal gRPC services all fail the same way.
- [ ] **The monitoring itself is verified against a deliberately near-expiry certificate** — an expiry check that has never fired is an expiry check you have no evidence works.

## 7. Protocol and cipher configuration {#protocol-and-cipher-configuration}

- [ ] **TLS 1.2 and TLS 1.3 are enabled and everything earlier is disabled** — TLS 1.0 and 1.1 are deprecated and fail modern compliance scans and payment requirements.
- [ ] **The cipher suite list is taken from a maintained reference configuration** — generate it rather than hand-curating, and record which compatibility level you chose and why.
- [ ] **Only forward-secret key exchanges are offered** — without forward secrecy, a future key compromise decrypts every session captured today.
- [ ] **Compression and renegotiation settings follow current guidance** — TLS compression is disabled, and client-initiated renegotiation is refused.
- [ ] **The configuration is verified by an external scanner and re-verified after every change** — configuration drift on load balancers is common and invisible until someone scans.
- [ ] **Session resumption is configured with rotating tickets** — static, never-rotated session ticket keys undermine forward secrecy for resumed sessions.
- [ ] **The same standard is applied to internal endpoints** — internal traffic is where obsolete protocol versions survive longest, precisely because nobody scans them.

## 8. OCSP stapling, HSTS, and hardening {#ocsp-stapling-hsts-and-hardening}

- [ ] **OCSP stapling is enabled and the stapled response is verified as present** — stapling removes a client-side round trip to the CA and stops the CA seeing your visitors, but a misconfigured stapler silently serves nothing.
- [ ] **Stapling failure is soft and monitored** — a server that hard-fails when the OCSP responder is unreachable turns a CA outage into your outage.
- [ ] **HSTS is enabled with a considered max-age** — start short, confirm every subdomain and service can serve HTTPS, then raise it, because HSTS cannot be withdrawn from clients that already cached it.
- [ ] **includeSubDomains is only enabled after every subdomain is confirmed HTTPS-capable** — this single directive has taken down internal tools that were only ever served over plain HTTP.
- [ ] **Preload submission is a deliberate decision with sign-off** — removal from preload lists takes months and reaches users only as browsers update.
- [ ] **HTTP requests redirect to HTTPS at the edge** — with a permanent redirect, and with no application content served over plain HTTP at all.
- [ ] **Certificate pinning, if used at all, has a documented backup pin and rollout plan** — pinning without a backup pin bricks clients when you rotate keys.

## 9. Incident path for an expired certificate {#incident-path-for-an-expired-certificate}

- [ ] **An emergency reissue procedure is documented and reachable without the affected service** — if your runbook lives behind the endpoint that just expired, you do not have a runbook.
- [ ] **The people with CA and registrar access are named in the on-call escalation** — issuance often needs an account only two people can reach.
- [ ] **The DNS-01 path is available as a fallback when the service is down** — HTTP-01 needs a working server, which is exactly what you do not have during an expiry incident.
- [ ] **Time to reissue and deploy has been measured, not assumed** — including CA issuance latency, propagation to every node, and the service reload.
- [ ] **Expired-certificate incidents always produce a follow-up on the automation** — the certificate is the symptom; the missing or unalerted renewal job is the cause, and replacing the certificate alone guarantees a repeat.
- [ ] **Rollback to the previous certificate is understood as usually impossible** — an expired certificate cannot be un-expired, which is why the prevention sections above are where the effort belongs.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Inventory and ownership | | | Pass / Pass with actions / Fail |
| Issuance and validation | | | Pass / Pass with actions / Fail |
| Key handling | | | Pass / Pass with actions / Fail |
| Chain and SAN correctness | | | Pass / Pass with actions / Fail |
| Automated renewal | | | Pass / Pass with actions / Fail |
| Expiry monitoring | | | Pass / Pass with actions / Fail |
| Protocol and cipher configuration | | | Pass / Pass with actions / Fail |
| OCSP stapling, HSTS, and hardening | | | Pass / Pass with actions / Fail |
| Incident path for an expired certificate | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with an owner, and treat any Fail in automated renewal or expiry monitoring as blocking.

## Related checklists

- [Secrets Management](/docs/security/secrets-management/)
- [Load Balancer Configuration](/docs/networking/load-balancer/)
- [DNS Migration](/docs/networking/dns-migration/)
- [Web Application Security](/docs/security/web-application-security/)
- [Production Readiness Review](/docs/devops/production-readiness/)

## References

- [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/)
- [NIST SP 800-52 Rev. 2 — Guidelines for TLS Implementations](https://csrc.nist.gov/pubs/sp/800/52/r2/final)
- [RFC 8446 — The Transport Layer Security (TLS) Protocol Version 1.3](https://www.rfc-editor.org/rfc/rfc8446)
- [RFC 6797 — HTTP Strict Transport Security (HSTS)](https://www.rfc-editor.org/rfc/rfc6797)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
