---
title: "Vendor Security Assessment"
description: "Verify that a third party handling your data or systems is assessed, contracted, monitored, and offboarded safely."
icon: "handshake"
weight: 840
toc: true
tags: ["third-party-risk", "vendor-management", "supply-chain", "compliance"]
---

Most organisations lose control of their data at the third hop: a vendor you assessed uses a sub-processor you never saw, running in a region you did not approve. Third-party assessment is worth doing properly only if the effort is proportionate to what the vendor can actually reach, and only if the assessment carries through into the contract, the monitoring, and the day the relationship ends. This checklist covers that full lifecycle for engineering and procurement teams.

{{< alert context="info" text="**Who runs this:** the security team with the business owner requesting the vendor, and procurement or legal for the contractual sections. **When:** before contract signature, on renewal, and whenever the vendor's scope of access or data materially changes." />}}

## 1. Intake and tiering {#intake-and-tiering}

- [ ] **A single intake path captures every new vendor before purchase** — assessment after the card has been charged is a negotiation you have already lost.
- [ ] **Tiering is driven by data sensitivity, access level, and business criticality** — a marketing tool holding email addresses and a payroll provider holding bank details do not deserve the same process.
- [ ] **The tiering criteria are written down and applied consistently** — otherwise the depth of assessment tracks the assessor's mood and the vendor's persistence.
- [ ] **Vendors with production system access are tiered highest regardless of data volume** — a monitoring agent with root on every host is a higher risk than a database of low-sensitivity records.
- [ ] **AI and data processing vendors are asked explicitly whether your data trains their models** — and the answer is captured in the assessment record, not only in a sales conversation.
- [ ] **Each vendor has a named internal business owner accountable for the relationship** — security cannot own a vendor it does not use.
- [ ] **The assessment depth per tier is defined with a target turnaround time** — an unbounded process is why teams route around it.

## 2. Questionnaire and evidence review {#questionnaire-and-evidence-review}

- [ ] **A standard questionnaire is used and sized to the tier** — a three-hundred-question set sent to a low-tier vendor produces copied answers nobody reads.
- [ ] **Answers are corroborated with evidence for anything material** — a claim of encryption at rest should be supported by a configuration statement, an architecture document, or an audit report reference.
- [ ] **Answers are compared against the vendor's public posture** — public documentation, status pages, trust centres, and breach history frequently contradict the questionnaire.
- [ ] **Follow-up questions are asked where an answer is vague** — not applicable and in progress are answers that need resolving before signature, not annotating.
- [ ] **The assessment records a risk decision, not just a score** — accept, accept with conditions, or reject, with the reasoning and the accepting owner named.
- [ ] **Conditional acceptances carry dated remediation commitments referenced in the contract** — a promise made in a questionnaire is unenforceable on its own.
- [ ] **The completed assessment is retained with a review date** — this is the artefact your own auditors will sample.

## 3. Certifications, reports, and their scope {#certifications-reports-and-their-scope}

- [ ] **Certificates and audit reports are obtained directly and checked for validity dates** — expired or lapsed certificates circulate in sales decks long after they stop being true.
- [ ] **The scope statement is read, not just the certificate front page** — an ISO 27001 certificate scoped to the vendor's corporate IT tells you nothing about the product you are buying.
- [ ] **SOC 2 reports are read past the opinion, including exceptions and management responses** — a report with exceptions and honest responses is more useful than a clean one with a narrow scope.
- [ ] **The report period is checked for gaps against your own reporting needs** — request a bridge letter where the period ends before your own audit window.
- [ ] **Complementary user entity controls in the report are extracted and assigned to owners on your side** — these are obligations the report places on you, and ignoring them undermines your own assurance.
- [ ] **Subservice organisations carved out of the vendor's report are identified** — the carve-out is the vendor telling you where their assurance stops.
- [ ] **Penetration test summaries are recent and cover the product, not the corporate website** — the scope and date matter more than the conclusion.

{{< alert context="warning" text="**Common mistake:** treating any certificate as a pass. A certificate proves that a defined scope met a standard on a date. Read the scope statement first, and if it does not cover the service you are buying, the certificate is evidence of nothing relevant." />}}

## 4. Technical and integration review {#technical-and-integration-review}

- [ ] **The data actually sent to the vendor is enumerated field by field** — teams routinely discover the integration ships far more than the assessment assumed.
- [ ] **Authentication into your systems uses least privilege with scoped, rotatable credentials** — a vendor with a permanent admin API key is an unmanaged access path.
- [ ] **Single sign-on and SCIM provisioning are available and enabled where the vendor holds user accounts** — without them, offboarding depends on the vendor's admin console and someone remembering.
- [ ] **Data residency and processing locations are confirmed against your requirements** — including support access, backups, and any secondary region.
- [ ] **Encryption in transit and at rest is verified, including key ownership** — customer-managed keys change the risk position materially for high-tier vendors.
- [ ] **Logging and audit trails are available to you, not only to the vendor** — an incident you cannot investigate on their platform is an incident you cannot close.
- [ ] **Any agent, browser extension, or code the vendor runs inside your environment is reviewed like your own code** — this is a direct supply-chain path into production.

## 5. Subprocessor and fourth-party chain {#subprocessor-and-fourth-party-chain}

- [ ] **A current subprocessor list is obtained and reviewed, not just acknowledged** — the list is where the actual data flow becomes visible.
- [ ] **Each subprocessor's purpose and location are recorded** — a hosting provider, an email relay, and an offshore support desk carry very different risks.
- [ ] **Change notification terms give you enough time to object meaningfully** — thirty days with a defined objection right is workable; a page that updates silently is not.
- [ ] **You are subscribed to the subprocessor change notification channel and someone reads it** — an unsubscribed notification right is no right at all.
- [ ] **Flow-down of your security and privacy obligations to subprocessors is contractually required** — otherwise your requirements stop at the first hop.
- [ ] **Concentration risk across vendors is understood** — several critical vendors on the same underlying platform means one outage takes all of them.

## 6. Contractual controls {#contractual-controls}

- [ ] **Security requirements are in the contract or an annexed security schedule, not only in the questionnaire** — assessment answers are usually not contractually binding.
- [ ] **Breach notification obligations have a defined deadline you can meet with your own regulators** — 72-hour regulatory clocks require a notification term measured in hours, not days.
- [ ] **A data processing agreement covers the roles, purposes, and instructions where personal data is involved** — with the international transfer mechanism named.
- [ ] **Audit or assurance rights are specified** — a right to receive the annual report and evidence of remediation is more realistic than an on-site audit right you will never exercise.
- [ ] **Data return and deletion obligations at termination are specified with a deadline and a certificate** — including backups and any derived data.
- [ ] **Service levels and support response times reflect how critical the vendor actually is** — and the remedy for missing them is meaningful rather than a token credit.
- [ ] **Liability and insurance terms are proportionate to the data at risk** — a cap at twelve months of fees is inadequate for a vendor holding your entire customer base.

## 7. Ongoing monitoring and reassessment {#ongoing-monitoring-and-reassessment}

- [ ] **Reassessment frequency is set per tier with a calendar owner** — annually for critical vendors, at renewal for the rest, and immediately on a material change.
- [ ] **New certifications and audit reports are collected on the vendor's cycle** — track their report period end date, not your own review date.
- [ ] **Public security incidents affecting vendors are monitored and trigger a review** — with a defined internal path for assessing whether your data was in scope.
- [ ] **Vendor-side changes that trigger reassessment are defined** — acquisition, region change, new subprocessor, product architecture change, or a change in the data you send.
- [ ] **Vendor availability and incidents are tracked against contractual service levels** — otherwise the SLA is never enforced.
- [ ] **The vendor inventory is reconciled against actual spend and actual integrations at least annually** — expense reports and OAuth grant lists both reveal vendors nobody assessed.
- [ ] **Critical vendor failure is covered in business continuity planning** — with an identified alternative or a documented acceptance of the outage.

## 8. Offboarding and termination {#offboarding-and-termination}

- [ ] **Offboarding is triggered by a defined event and owned by the business owner** — contract expiry, migration, or an unrenewed trial should all start the same process.
- [ ] **Your data is exported in a usable format before access is terminated** — the order matters, and teams regularly discover this the wrong way round.
- [ ] **Deletion is requested in writing and a confirmation or certificate is retained** — including backups, with the vendor's stated backup expiry window recorded.
- [ ] **All credentials issued to the vendor are revoked** — API keys, service accounts, VPN access, SSH keys, and any OAuth application grant.
- [ ] **Vendor accounts in your systems and your accounts in theirs are both closed** — the second direction is the one that gets forgotten and keeps data reachable.
- [ ] **Integrations, webhooks, and DNS or email records pointing at the vendor are removed** — a dangling subdomain pointing at a decommissioned vendor is a takeover risk.
- [ ] **The vendor inventory and the record of processing are updated to reflect the termination** — a stale inventory undermines every downstream assurance activity.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Intake and tiering | | | Pass / Pass with actions / Fail |
| Questionnaire and evidence review | | | Pass / Pass with actions / Fail |
| Certifications, reports, and their scope | | | Pass / Pass with actions / Fail |
| Technical and integration review | | | Pass / Pass with actions / Fail |
| Subprocessor and fourth-party chain | | | Pass / Pass with actions / Fail |
| Contractual controls | | | Pass / Pass with actions / Fail |
| Ongoing monitoring and reassessment | | | Pass / Pass with actions / Fail |
| Offboarding and termination | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated condition of approval with a named owner, and reference the conditions in the contract so they remain enforceable after signature.

## Related checklists

- [GDPR Readiness](/docs/compliance/gdpr-readiness/)
- [SOC 2 Audit Readiness](/docs/compliance/soc2-audit-readiness/)
- [Secrets Management](/docs/security/secrets-management/)
- [Cloud Security](/docs/security/cloud-security/)
- [IT Asset Management](/docs/itsm/it-asset-management/)

## References

- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [CIS Critical Security Controls](https://www.cisecurity.org/controls)
- [ISO/IEC 27001 — Information security management](https://www.iso.org/standard/27001)
- [AICPA and CIMA — SOC suite of services](https://www.aicpa-cima.com/resources/landing/system-and-organization-controls-soc-suite-of-services)
- [NCSC — Supply chain security guidance](https://www.ncsc.gov.uk/collection/supply-chain-security)
