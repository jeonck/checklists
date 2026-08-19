---
title: "ISO/IEC 27001 ISMS"
description: "Verify that an information security management system is complete, operating, and evidenced before the certification audit."
icon: "verified"
weight: 820
toc: true
tags: ["iso27001", "isms", "compliance", "audit"]
---

An ISO/IEC 27001 certification audit does not test whether your security is good. It tests whether you have a management system that identifies risk, treats it deliberately, checks itself, and corrects itself — and whether you can prove all of that with records. Most first-time failures are not control failures; they are missing evidence that the management cycle actually ran. This checklist walks the clauses and the Annex A implementation work in the order an auditor tends to open them.

{{< alert context="info" text="**Who runs this:** the ISMS manager, with control owners from engineering, IT, HR, and legal. **When:** three to six months before the Stage 1 audit, and again before each surveillance visit." />}}

{{< alert context="warning" text="This checklist supports but does not replace advice from a qualified auditor or information security professional. Certification decisions rest with an accredited certification body, and nothing here guarantees a particular audit outcome." />}}

## 1. Scope, context, and interested parties

- [ ] **The ISMS scope statement names the services, locations, and organisational units included** — and the exclusions are stated explicitly, because an ambiguous scope is the fastest route to a Stage 1 delay.
- [ ] **Interfaces and dependencies at the scope boundary are described** — shared corporate IT, a parent company's network, and outsourced development all sit on the boundary and will be probed.
- [ ] **Internal and external issues affecting the ISMS are documented and reviewed** — regulatory change, cloud dependency, remote working, and market pressure are the usual entries.
- [ ] **Interested parties and their requirements are identified** — customers, regulators, employees, and investors, with the specific requirement each imposes rather than a generic list.
- [ ] **The scope is credible against the certificate you intend to advertise** — scoping out the systems your customers actually care about produces a certificate that fails their vendor review.
- [ ] **Top management commitment is evidenced, not asserted** — a signed policy, resourced roles, and attendance records at management review are what the auditor will look for.

## 2. Risk assessment and treatment methodology

- [ ] **A documented risk assessment methodology defines the criteria for acceptance and the scales used** — repeatability is the requirement, so two assessors following it should reach comparable results.
- [ ] **Risk owners are named individuals with the authority to accept risk** — a team name is not a risk owner.
- [ ] **The asset, threat, and vulnerability inventory behind the assessment is current** — reconcile it against the asset register and the cloud inventory rather than against last year's spreadsheet.
- [ ] **Every identified risk has a treatment decision** — modify, retain, avoid, or share, each with a rationale recorded.
- [ ] **The risk treatment plan lists actions, owners, and target dates and is actively tracked** — overdue treatment actions with no revised date are one of the most common minor nonconformities.
- [ ] **Residual risk is formally accepted by the risk owner** — with a dated record, not implied by the absence of objection.
- [ ] **The assessment has been re-run within the last twelve months and after any significant change** — a merger, a new product line, or a major architecture change all trigger a re-run.

## 3. Statement of Applicability and Annex A controls

- [ ] **The Statement of Applicability covers every Annex A control with an applicability decision and justification** — including justifications for exclusions, which is where auditors concentrate.
- [ ] **Each applicable control names an owner and points to the evidence of implementation** — the SoA is the index the auditor navigates by, so make it useful.
- [ ] **The SoA is aligned to the current version of the standard** — the 2022 revision restructured Annex A into four themes and added controls such as threat intelligence, cloud services, and secure coding.
- [ ] **Newer controls are genuinely implemented, not mapped to something old** — data leakage prevention, configuration management, and information deletion are frequently claimed and rarely evidenced.
- [ ] **Technical controls are verified in the live environment, not from a design document** — sample actual firewall rules, IAM policies, endpoint configurations, and logging destinations.
- [ ] **Controls inherited from cloud providers are separated from controls you operate** — the shared responsibility split must be written down, or you will be asked to evidence something you do not control.
- [ ] **Control effectiveness has measurable indicators where practical** — patch compliance rate, mean time to revoke access, phishing simulation failure rate.

## 4. Documented information and policy management

- [ ] **Every policy has a named owner, an approval record, and a review date within the last twelve months** — an unreviewed policy signals a dormant management system more clearly than any control gap.
- [ ] **Document control covers versioning, approval, distribution, and withdrawal of superseded copies** — including the stale PDF still sitting on the shared drive.
- [ ] **Staff acknowledgement of policies is recorded with dates and coverage is reportable** — including contractors and new joiners within their first weeks.
- [ ] **Procedures describe what people actually do** — interview a practitioner and compare; the auditor will.
- [ ] **Records required by the standard are retained and retrievable within the audit window** — training records, audit reports, management review minutes, corrective actions, and risk acceptances.
- [ ] **Access to ISMS documentation is controlled and available to those who need it** — a policy set nobody can find is not implemented.

## 5. Competence, awareness, and operational planning

- [ ] **Roles with ISMS responsibilities have defined competence requirements** — and evidence that the holder meets them through qualification, training, or experience.
- [ ] **Security awareness training is delivered at least annually with completion tracked per person** — and the content reflects the risks in your risk register, not a generic catalogue.
- [ ] **Role-specific training exists for developers, administrators, and privileged users** — a single company-wide module will not satisfy a competence question for a database administrator.
- [ ] **Communication about the ISMS is planned** — who communicates what, to whom, and when, both internally and to customers or regulators.
- [ ] **Outsourced processes within scope are identified and controlled** — managed service providers, offshore development, and payroll processing are all in play.
- [ ] **Changes to the ISMS are made in a planned way with consequences considered** — unplanned change is explicitly called out by the standard.

## 6. Monitoring, measurement, and internal audit

- [ ] **What is monitored, how, by whom, and how often is defined for each measure** — a metrics dashboard without a defined method fails the clause even if the numbers are good.
- [ ] **An internal audit programme covers the whole ISMS over a defined cycle and is documented** — with the cycle length justified against risk and previous results.
- [ ] **Internal auditors are objective and do not audit their own work** — using the person who wrote the procedure is a finding regardless of how well they audit.
- [ ] **Internal audit reports record findings, evidence sampled, and conclusions** — a report saying everything was satisfactory with no evidence trail is treated as no audit at all.
- [ ] **Internal audit has actually found something** — an audit programme that never raises a nonconformity invites the certification body to question its rigour.
- [ ] **Findings are tracked to closure with verification of effectiveness** — closing a finding because the action was completed is not the same as verifying it worked.
- [ ] **Technical assurance activities feed the ISMS** — vulnerability scanning, penetration testing, and access reviews should appear as inputs, not run in a separate universe.

{{< alert context="danger" text="**Blocking:** a Stage 2 audit cannot succeed without at least one completed internal audit cycle and one completed management review covering the current scope. If either is missing, move the audit date rather than hoping the auditor overlooks it." />}}

## 7. Management review and continual improvement

- [ ] **Management review has been held within the planned interval and is minuted** — with attendees recorded, since top management presence is the point of the clause.
- [ ] **Every required input is on the agenda and visible in the minutes** — status of previous actions, changes in issues and interested parties, performance measures, audit results, risk assessment status, and improvement opportunities.
- [ ] **Outputs include decisions on improvements and resources** — a review that produces no decisions is a status meeting.
- [ ] **Nonconformities are recorded with root cause analysis, not just a fix** — the standard asks whether the cause could produce similar nonconformities elsewhere.
- [ ] **Corrective actions have owners, dates, and evidence of effectiveness review** — the effectiveness check is the step teams skip most often.
- [ ] **Improvement suggestions from staff and incidents reach the ISMS** — a feedback path that only carries audit findings misses most of the useful signal.
- [ ] **Trends across incidents, findings, and metrics are analysed rather than reported item by item** — this is what distinguishes a maturing system from a compliant one.

## 8. Certification audit stages and logistics

- [ ] **The certification body is accredited and the accreditation has been verified** — an unaccredited certificate will be rejected by the customers you obtained it for.
- [ ] **Stage 1 documentation is assembled as a coherent pack** — scope, policy, risk methodology, risk assessment, treatment plan, SoA, internal audit report, and management review minutes.
- [ ] **Stage 1 findings have been closed before Stage 2 begins** — arriving at Stage 2 with open Stage 1 observations wastes the audit days you paid for.
- [ ] **Evidence for the audit period exists across the full period, not just the last month** — sampling will reach back, and a burst of activity in the final weeks is visible.
- [ ] **Control owners are briefed, available, and able to demonstrate their control live** — screen sharing into the actual console beats a screenshot every time.
- [ ] **A guide is assigned to the auditor and answers stay within the question asked** — volunteering unprompted detail is how audits expand.
- [ ] **The surveillance and recertification cycle is planned and resourced** — certification is a three-year cycle with annual surveillance, and the second year is where systems quietly lapse.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Scope, context, and interested parties | | | Pass / Pass with actions / Fail |
| Risk assessment and treatment methodology | | | Pass / Pass with actions / Fail |
| Statement of Applicability and Annex A controls | | | Pass / Pass with actions / Fail |
| Documented information and policy management | | | Pass / Pass with actions / Fail |
| Competence, awareness, and operational planning | | | Pass / Pass with actions / Fail |
| Monitoring, measurement, and internal audit | | | Pass / Pass with actions / Fail |
| Management review and continual improvement | | | Pass / Pass with actions / Fail |
| Certification audit stages and logistics | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated nonconformity or improvement item in the ISMS tracker, so the corrective action process itself becomes part of the audit evidence.

## Related checklists

- [SOC 2 Audit Readiness](/docs/compliance/soc2-audit-readiness/)
- [GDPR Readiness](/docs/compliance/gdpr-readiness/)
- [Vendor Security Assessment](/docs/compliance/vendor-security-assessment/)
- [Incident Response](/docs/security/incident-response/)
- [Change Management](/docs/itsm/change-management/)

## References

- [ISO/IEC 27001 — Information security management](https://www.iso.org/standard/27001)
- [ISO — Information security, cybersecurity and privacy protection](https://www.iso.org/isoiec-27001-information-security.html)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [CIS Critical Security Controls](https://www.cisecurity.org/controls)
