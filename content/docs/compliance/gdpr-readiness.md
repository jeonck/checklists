---
title: "GDPR Readiness"
description: "Verify that personal data processing has a lawful basis, working rights workflows, and evidence a regulator would accept."
icon: "privacy_tip"
weight: 810
toc: true
tags: ["gdpr", "privacy", "compliance", "data-protection"]
---

GDPR compliance is not a document exercise. A supervisory authority asking questions after a complaint will want to see the record of processing, the retention job that actually deletes data, the ticket trail behind a subject access request, and the transfer assessment behind the analytics vendor in the United States. This checklist covers the engineering and operational work that makes those artefacts real rather than aspirational, and is aimed at teams that build and run the systems where personal data lives.

{{< alert context="info" text="**Who runs this:** the data protection officer or privacy lead, together with the engineering owners of each system that processes personal data. **When:** annually, and whenever a new processing activity, product, or major vendor is introduced." />}}

{{< alert context="warning" text="This checklist supports but does not replace advice from a qualified privacy professional or legal counsel. Interpretations of lawful basis, transfer mechanisms, and controller/processor roles are fact-specific, and the final judgement belongs to your DPO and legal advisers." />}}

## 1. Lawful basis and transparency

- [ ] **Every processing activity has one identified lawful basis recorded before processing starts** — picking a basis retrospectively, or listing several in case one fails, is a finding an auditor will pick up immediately.
- [ ] **Legitimate interests assessments exist in writing for every activity relying on that basis** — with the purpose, the necessity argument, and the balancing test against the data subject's rights documented and dated.
- [ ] **Consent, where used, is freely given, granular, and recorded with evidence** — store what was shown, which version of the notice, the timestamp, and the mechanism, so you can reproduce the consent event years later.
- [ ] **Withdrawing consent is as easy as giving it and takes effect in downstream systems** — a preference toggle that stops the marketing tool but not the warehouse export is not a withdrawal.
- [ ] **Special category data is identified and has an Article 9 condition on top of the lawful basis** — health, biometric, and trade union data cannot ride on legitimate interests alone.
- [ ] **The privacy notice matches what the systems actually do** — compare it against the record of processing and the outbound data flows, not against what the product team intended.
- [ ] **Children's data and age assurance are addressed where the service may reach minors** — including whether parental consent is required in each jurisdiction you serve.

## 2. Records of processing and data mapping

- [ ] **An Article 30 record of processing exists and names a responsible owner per activity** — an unowned record goes stale within a quarter and is worthless at the moment it is requested.
- [ ] **Each record lists purposes, data categories, data subject categories, recipients, transfers, and retention periods** — these are the fields the regulation names, and gaps in them are the fastest finding an inspector can raise.
- [ ] **The record is reconciled against reality at least annually** — sample a few activities and trace the actual database tables, log sinks, and third-party integrations that hold the data.
- [ ] **Personal data is classified in the data catalogue or schema metadata** — column-level tagging is what makes rights fulfilment and retention automatable rather than manual.
- [ ] **Shadow copies are mapped as well as primary stores** — analytics warehouses, backups, log aggregators, support tickets, CRM exports, and spreadsheets on shared drives all hold personal data.
- [ ] **Controller and processor roles are determined per activity and written down** — joint controllership with a partner requires an Article 26 arrangement, and getting the role wrong invalidates the contract you signed.

## 3. Data subject rights workflows

- [ ] **There is a single documented intake channel for rights requests with a named triage owner** — requests arriving by support chat, email, or postal letter must all reach the same queue.
- [ ] **The one-month response deadline is tracked in a system, not in someone's calendar** — with the extension to three months used only for genuinely complex requests and communicated within the first month.
- [ ] **Identity verification is proportionate and documented** — demanding a passport scan for a routine access request creates its own compliance problem, while verifying nothing invites disclosure to an impersonator.
- [ ] **Access requests can be fulfilled from every system holding the subject's data** — run a test request end to end and time it; if it takes three engineers a week, the process will fail under load.
- [ ] **Erasure propagates to backups, replicas, search indexes, and processors with a documented approach** — an auditor accepts a documented backup rotation window; they do not accept silence about backups.
- [ ] **Rectification, restriction, portability, and objection each have a defined procedure** — teams typically build access and erasure and then improvise the rest under time pressure.
- [ ] **Portability exports use a structured, commonly used, machine-readable format** — a PDF of screenshots does not satisfy Article 20.
- [ ] **Refusals are logged with the reason and the exemption relied on** — and the requester is told about their right to complain to a supervisory authority.

{{< alert context="danger" text="**Blocking:** if a subject access request cannot be fulfilled without a bespoke engineering project each time, you do not have a rights workflow. Rehearse one end to end and measure the elapsed time before declaring readiness." />}}

## 4. Data minimisation and retention

- [ ] **Each field collected maps to a stated purpose** — review new forms and API payloads for fields collected because they might be useful later.
- [ ] **Retention periods are defined per data category and justified, not chosen for convenience** — indefinite retention is a decision that needs an argument behind it.
- [ ] **Retention is enforced by an automated job with monitored output** — a policy document that says ninety days while the table holds seven years of rows is worse than no policy.
- [ ] **Deletion covers derived data** — aggregates, embeddings, model training sets, and feature stores can re-identify individuals long after the source row is gone.
- [ ] **Pseudonymisation and anonymisation claims are tested, not asserted** — if the mapping table still exists inside your control, the data is pseudonymous and still personal data.
- [ ] **Non-production environments do not hold live personal data** — or, where they must, the masking process is documented and its output is spot-checked.
- [ ] **Log and telemetry pipelines are reviewed for personal data** — request bodies, full URLs with query parameters, and error payloads are the usual leak paths.

## 5. International transfers

- [ ] **Every transfer of personal data outside the EEA is identified, including remote access by support staff** — an engineer in a third country with production database access is a transfer.
- [ ] **Each transfer has a valid mechanism recorded** — adequacy decision, standard contractual clauses with the correct module, binding corporate rules, or a documented derogation.
- [ ] **Transfer impact assessments exist for SCC-based transfers** — covering the destination country's surveillance laws and the supplementary technical measures you rely on.
- [ ] **Supplementary measures are technical where possible** — encryption with keys held in the EEA is defensible; a contractual promise alone generally is not.
- [ ] **Cloud region configuration is verified against the transfer position** — check the actual resource regions, replication settings, backup destinations, and any managed service that processes data in a different region.
- [ ] **Sub-processor locations are tracked and change notifications are monitored** — vendors move regions and add sub-processors with a fixed notice period you must actually be reading.

## 6. Processor contracts and vendor obligations

- [ ] **Every processor is under an Article 28 agreement signed before data flows** — including the small tools procured on a corporate card outside the normal process.
- [ ] **The agreement contains all Article 28(3) elements** — subject matter, duration, purpose, instructions, confidentiality, security, sub-processing, rights assistance, deletion or return, and audit rights.
- [ ] **Sub-processor authorisation and the objection window are defined and workable** — a five-day notice period you cannot act on is a term you should renegotiate.
- [ ] **Processor security is verified with evidence, not a self-declaration** — a current certification with a scope statement, a penetration test summary, or a completed assessment.
- [ ] **Breach notification obligations flow through to processors with a deadline you can meet** — you have 72 hours from your own awareness, so a processor with a seven-day notice term makes that impossible.
- [ ] **End-of-contract deletion or return is exercised and evidenced** — request the deletion certificate rather than assuming it happened.

## 7. Data protection impact assessments

- [ ] **A documented screening test decides when a DPIA is required** — applied to every new processing activity, not only to the ones someone felt uneasy about.
- [ ] **DPIAs cover systematic monitoring, large-scale special category processing, and automated decision-making with legal effect** — these are the categories most likely to attract regulator attention.
- [ ] **Each DPIA describes the processing, assesses necessity and proportionality, identifies risks to individuals, and records mitigations** — risks to the individual, not risks to the business.
- [ ] **The DPO is consulted and their advice recorded, including where it was not followed** — the record of the disagreement is itself part of the accountability evidence.
- [ ] **Residual high risk after mitigation triggers prior consultation with the supervisory authority** — the fact that this is rare does not mean the trigger can be undefined.
- [ ] **DPIAs are revisited when the processing changes materially** — a new data source, a new purpose, or a new automated decision restarts the assessment.

## 8. Personal data breach response

- [ ] **A personal data breach is defined in the incident process and is distinguishable from a security incident** — availability and integrity incidents count, not just confidentiality.
- [ ] **The 72-hour clock starts at awareness and the process makes that point explicit** — awareness usually begins with a first responder, not with the moment leadership is briefed.
- [ ] **The notification decision path names who decides and who signs** — with a documented deputy, because breaches do not respect holidays.
- [ ] **Draft notification templates for the supervisory authority and for data subjects exist and are pre-reviewed** — writing them under time pressure produces the errors that turn a breach into an enforcement case.
- [ ] **An internal breach register records every incident, including ones not notified, with the reasoning** — Article 33(5) requires this and it is one of the first things requested during an investigation.
- [ ] **A breach exercise has been run in the last twelve months involving privacy, security, legal, and communications** — measure whether the 72-hour path actually completes.
- [ ] **Processors know how to reach you out of hours** — and the contact detail in the contract is a monitored channel, not a departed employee's inbox.

## 9. Privacy by design in the SDLC

- [ ] **A privacy review is a gate in the design process for features touching personal data** — attached to the design review, not bolted on before launch.
- [ ] **Default settings are the most privacy-protective option** — Article 25(2) is about defaults, and a permissive default that users must find and disable fails it.
- [ ] **New personal data fields require a recorded purpose and retention period before the schema change merges** — put the check in the migration review, where it is cheap.
- [ ] **Automated decision-making with significant effect has a human review path and an explanation** — and users are told the logic involved, in plain language.
- [ ] **Third-party scripts and trackers on web properties are inventoried and gated by consent** — tag managers are a common route for undisclosed data flows to appear without a code change.
- [ ] **Data flows in the architecture diagram show personal data crossing trust and jurisdictional boundaries** — this is what makes transfer and processor questions visible during design.
- [ ] **Privacy training is completed by engineers and product staff and the completion is evidenced** — with role-specific content, since a generic annual module teaches nobody how to design a retention job.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Lawful basis and transparency | | | Pass / Pass with actions / Fail |
| Records of processing and data mapping | | | Pass / Pass with actions / Fail |
| Data subject rights workflows | | | Pass / Pass with actions / Fail |
| Data minimisation and retention | | | Pass / Pass with actions / Fail |
| International transfers | | | Pass / Pass with actions / Fail |
| Processor contracts and vendor obligations | | | Pass / Pass with actions / Fail |
| Data protection impact assessments | | | Pass / Pass with actions / Fail |
| Personal data breach response | | | Pass / Pass with actions / Fail |
| Privacy by design in the SDLC | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with a named owner, and have the DPO confirm the residual risk position before the review is closed.

## Related checklists

- [Vendor Security Assessment](/docs/compliance/vendor-security-assessment/)
- [ISO/IEC 27001 ISMS](/docs/compliance/iso27001-isms/)
- [Incident Response](/docs/security/incident-response/)
- [Backup and Recovery](/docs/operations/backup-and-recovery/)
- [Database Schema Migration](/docs/development/database-schema-migration/)

## References

- [GDPR full text (gdpr-info.eu)](https://gdpr-info.eu/)
- [Regulation (EU) 2016/679 on EUR-Lex](https://eur-lex.europa.eu/eli/reg/2016/679/oj)
- [European Data Protection Board](https://www.edpb.europa.eu/)
- [European Commission — Data protection](https://commission.europa.eu/law/law-topic/data-protection_en)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
