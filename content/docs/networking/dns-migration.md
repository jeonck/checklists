---
title: "DNS Migration"
description: "Move a zone between DNS providers or nameservers without dropping traffic or breaking DNSSEC."
icon: "dns"
weight: 720
toc: true
tags: ["dns", "networking", "migration", "cutover"]
---

DNS migrations are unforgiving because the mistake is invisible at the moment you make it and becomes visible to users hours later, spread unevenly across the internet as caches expire. The work that determines whether a migration is boring happens days before the cutover: lowering TTLs, inventorying every record, and proving the new provider answers identically to the old one. This checklist is ordered in time, from roughly a week out to a week after.

{{< alert context="info" text="**Who runs this:** the team that owns the zone, with the registrar account holder available live during the cutover. **When:** section 1 at least 48 hours before the cutover, sections 4 to 6 during it, section 8 no earlier than 48 hours after." />}}

## 1. TTL reduction, at least 48 hours ahead

- [ ] **Every record being moved has had its TTL lowered to 300 seconds or less** — a record still cached at its old 86400 second value cannot be rolled back quickly, so the TTL reduction is the thing that buys you a fast undo.
- [ ] **The TTL reduction was applied at least one full old-TTL period before the cutover** — lowering a 24 hour TTL two hours before cutover achieves nothing, because resolvers holding the old value keep it for the old duration.
- [ ] **The NS record TTL and the SOA minimum TTL were lowered as well** — these govern negative caching and delegation caching, and are commonly missed because they are not in the records list people think of.
- [ ] **The registrar-side delegation TTL is understood and documented** — TLD delegation TTLs are set by the registry, typically one or two days, and you cannot lower them, so plan the cutover around that constraint rather than assuming you control it.
- [ ] **A calendar entry exists to restore TTLs after the migration** — low TTLs multiply query volume and cost, and left in place they become the permanent default nobody remembers to change.
- [ ] **The current TTL values were recorded before lowering them** — you need the original values to restore, and reconstructing them from memory produces a zone that drifts a little more with every migration.

## 2. Record inventory and parity

- [ ] **A complete zone export exists from the current provider** — prefer a real AXFR or the provider's zone export rather than reading the web console, which frequently hides provider-specific records.
- [ ] **Provider-proprietary record types are identified and mapped to a portable equivalent** — ALIAS, ANAME, CNAME-at-apex, and weighted or geo-routed records are not standard and rarely survive a lift-and-shift.
- [ ] **Email records are inventoried explicitly** — MX, SPF TXT, DKIM selectors, DMARC, and any provider verification TXT records, because losing a DKIM selector silently sends legitimate mail to spam folders.
- [ ] **Domain ownership verification records are listed** — Google, Microsoft, and certificate authority verification TXT records get dropped in migrations and only surface when a renewal fails weeks later.
- [ ] **Subdomain delegations are captured** — any NS record inside the zone delegating a child zone elsewhere must be recreated, or that whole subtree goes dark.
- [ ] **CAA records are carried over** — a missing CAA record is permissive and therefore harmless, but a wrong one blocks certificate issuance at exactly the wrong moment.
- [ ] **The new provider's zone is diffed against the export record by record** — automate the comparison; a manual eyeball over a zone of any size will miss entries.

```bash
# compare the two providers answer-for-answer, per record name
for name in $(cut -f1 records.txt); do
  a=$(dig +short "$name" ANY @ns1.old-provider.example)
  b=$(dig +short "$name" ANY @ns1.new-provider.example)
  [ "$a" = "$b" ] || echo "MISMATCH $name"
done
```

{{< alert context="warning" text="**Common mistake:** exporting only A and CNAME records. The records that cause the worst post-migration incidents are TXT records for SPF and DKIM, and NS records delegating subdomains, because nothing fails immediately and nobody connects the eventual failure to the migration." />}}

## 3. Pre-cutover validation on the new provider

- [ ] **The new nameservers answer authoritatively for the zone before delegation changes** — query them directly by IP and confirm the AA flag is set and the SOA serial is current.
- [ ] **Responses match the old provider for a representative sample of names** — including the apex, www, wildcard behaviour, and at least one name that should not exist.
- [ ] **NXDOMAIN behaviour is identical for names that do not exist** — a provider that returns a wildcard answer or a synthesised response where the old one returned NXDOMAIN will break mail routing and service discovery.
- [ ] **The zone's own NS record set matches the delegation you intend to publish** — a mismatch between the parent delegation and the in-zone NS set produces lame delegation warnings and inconsistent resolver behaviour.
- [ ] **The SOA record values are deliberate** — refresh, retry, expire, and minimum, with expire long enough that a provider outage does not cause secondaries to stop answering.
- [ ] **The zone passes an external validity check** — run a third-party zone analyser and resolve every warning or record why it is acceptable.
- [ ] **Any traffic-management behaviour is reproduced and tested** — health-checked failover, latency routing, or weighted answers behave differently between providers and must be verified from more than one vantage point.

## 4. DNSSEC handling

- [ ] **Whether the zone is currently signed has been checked at the parent, not just at the provider** — the presence of a DS record in the parent zone is what makes validation mandatory, and it is the thing that breaks the domain if it goes stale.
- [ ] **A migration path for DNSSEC is chosen explicitly** — either both providers support multi-signer operation with shared keys, or you go insecure temporarily by removing the DS record and waiting out its TTL before switching.
- [ ] **If going insecure temporarily, the DS record was removed and its TTL fully elapsed before the nameserver change** — switching nameservers while a DS record still points at the old key makes every validating resolver return SERVFAIL, which is a total outage for a large fraction of users.
- [ ] **The new DS record is published to the parent only after the new provider is authoritative and its DNSKEY is live** — publishing early produces the same SERVFAIL outage in the opposite direction.
- [ ] **Validation is confirmed from a validating resolver after the DS is published** — check the AD flag on a response and confirm a deliberately broken name fails as expected.
- [ ] **Key rollover responsibility at the new provider is documented** — who or what rotates the KSK, and whether DS updates to the registrar are automated through CDS/CDNSKEY or need a human.

{{< alert context="danger" text="**Blocking:** never change nameservers while a DS record for the old provider's key is still published in the parent zone. Validating resolvers will fail closed, and the domain is hard down for those users until the DS TTL expires." />}}

## 5. Registrar and nameserver change

- [ ] **Access to the registrar account is confirmed working before the window** — including multi-factor authentication and the person who holds it being present, not merely reachable.
- [ ] **Registrar lock status is checked and the domain expiry date is comfortably far away** — a migration is a bad time to discover the domain expires next week.
- [ ] **The registrant, admin, and technical contact email addresses are deliverable** — verification mail sent to a mailbox nobody reads stalls the change mid-window.
- [ ] **The old provider's zone is left in place and serving, unchanged, during the cutover** — deleting it is the single most common way to turn a reversible migration into an outage.
- [ ] **The nameserver change is made as a complete replacement of the NS set** — mixed old and new nameservers means resolvers get answers from whichever they picked, which makes any inconsistency intermittent and nearly impossible to debug.
- [ ] **The exact time of the delegation change is recorded** — every subsequent propagation observation is meaningless without a start time to measure from.

## 6. Propagation verification

- [ ] **The parent zone delegation is queried directly to confirm the change landed** — query a TLD nameserver for the NS records rather than trusting the registrar console, which reflects intent rather than published state.
- [ ] **Resolution is checked from multiple independent public resolvers and geographies** — your office resolver having the new answer says nothing about the rest of the world.
- [ ] **Query volume at the old provider is watched as it decays** — the old provider's traffic graph is the only honest measure of how many clients are still using the previous answers.
- [ ] **Application-level health is verified, not just DNS answers** — a correct A record pointing at a host with no matching virtual host or certificate is still an outage.
- [ ] **Mail flow is tested end to end after the change** — send and receive a real message, and check that SPF, DKIM, and DMARC all pass at the receiving end.
- [ ] **A named rollback trigger and owner exist for the propagation window** — reverting the NS set at the registrar, with the old zone still intact, is the rollback, and it costs one delegation TTL.

## 7. Post-cutover stabilisation

- [ ] **The old provider's zone stays live and in sync for a defined period** — at least a week, long enough for stragglers with broken or hard-coded caching to be flushed out.
- [ ] **Any change made to the zone during that period is applied to both providers** — a divergence between them turns residual old traffic into inconsistent behaviour.
- [ ] **Certificate issuance and renewal is tested against the new provider** — ACME DNS-01 challenges depend on API credentials and propagation speed at the new provider, and a renewal that fails silently surfaces sixty days later as an expired certificate.
- [ ] **Monitoring for the zone is repointed and expanded** — alert on NXDOMAIN rates, SERVFAIL rates, and authoritative response failures, not just on whether one name resolves.
- [ ] **Automation and infrastructure-as-code definitions are updated to the new provider** — a Terraform state still describing the old provider will helpfully revert your migration on the next apply.
- [ ] **Domain expiry and DNSSEC key expiry alerts are configured with a long lead time** — thirty days minimum, delivered to a team address rather than an individual.

## 8. TTL restoration and closure

- [ ] **TTLs are restored to their normal values only after the stabilisation period ends** — typically 3600 seconds or higher for stable records, using the values recorded in section 1.
- [ ] **Records that genuinely need agility keep a low TTL deliberately** — failover targets and endpoints under active change, with the reason noted so a future cleanup does not raise them.
- [ ] **The old provider's zone is decommissioned only after a final export is archived** — keep the export, because it is the last complete record of the pre-migration state.
- [ ] **The old provider's account and API credentials are revoked** — stale DNS API credentials are a high-value target since they permit domain takeover and certificate issuance.
- [ ] **The runbook and zone documentation are updated with the new provider's specifics** — who has access, how records are changed, and how DNSSEC keys are managed.
- [ ] **The migration is reviewed for anything that surprised the team** — DNS surprises repeat, and the notes are worth more than the change record.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| TTL reduction | | | Pass / Pass with actions / Fail |
| Record inventory and parity | | | Pass / Pass with actions / Fail |
| Pre-cutover validation | | | Pass / Pass with actions / Fail |
| DNSSEC handling | | | Pass / Pass with actions / Fail |
| Registrar and nameserver change | | | Pass / Pass with actions / Fail |
| Propagation verification | | | Pass / Pass with actions / Fail |
| Post-cutover stabilisation | | | Pass / Pass with actions / Fail |
| TTL restoration and closure | | | Pass / Pass with actions / Fail |

The first four sections must be a full Pass before the delegation is changed; the remainder may close with a dated follow-up ticket and a named owner.

## Related checklists

- [TLS Certificate Management](/docs/networking/tls-certificate/)
- [Network Change](/docs/networking/network-change/)
- [Load Balancer Configuration](/docs/networking/load-balancer/)
- [Cloud Migration](/docs/cloud/cloud-migration/)
- [Release Day](/docs/devops/release-day/)

## References

- [RFC 1034 — Domain Names: Concepts and Facilities](https://www.rfc-editor.org/rfc/rfc1034)
- [RFC 1035 — Domain Names: Implementation and Specification](https://www.rfc-editor.org/rfc/rfc1035)
- [RFC 6781 — DNSSEC Operational Practices, Version 2](https://www.rfc-editor.org/rfc/rfc6781)
- [RFC 8901 — Multi-Signer DNSSEC Models](https://www.rfc-editor.org/rfc/rfc8901)
- [RFC 7208 — Sender Policy Framework (SPF) for Authorizing Use of Domains in Email](https://www.rfc-editor.org/rfc/rfc7208)
