---
title: "SOC 2 Audit Readiness"
description: "Verify that controls, narratives, and evidence will survive a SOC 2 examination before the observation window opens."
icon: "assignment_turned_in"
weight: 830
toc: true
tags: ["soc2", "audit", "compliance", "evidence"]
---

A SOC 2 Type II report is an opinion on whether your controls operated effectively over a period, and the auditor forms that opinion by sampling evidence from across the window. The window is unforgiving: a control that started working in month five leaves four months of exceptions that cannot be repaired afterwards. This checklist is about getting the control set, the narratives, and the evidence pipeline in place before the clock starts, and keeping them intact while it runs.

{{< alert context="info" text="**Who runs this:** the compliance owner, with control owners in engineering, IT, HR, and security, and a readiness reviewer independent of the control operators. **When:** at least one quarter before the Type II observation window opens, and monthly while it is running." />}}

## 1. Scope and trust services criteria selection {#scope-and-trust-services-criteria-selection}

- [ ] **The system description boundary names the products, environments, and supporting infrastructure in scope** — a boundary drawn loosely will pull unrelated systems into sampling.
- [ ] **Security is included and the additional criteria are chosen for a reason** — availability, confidentiality, processing integrity, and privacy each add real evidence burden, so add them because a customer contract requires them.
- [ ] **Type I versus Type II is decided deliberately** — a Type I proves design at a point in time and buys credibility while the Type II window runs; going straight to Type II with immature controls produces a report full of exceptions.
- [ ] **The observation window length is agreed with the auditor** — three months is common for a first report, twelve for a mature one, and gaps between consecutive reports get flagged by customers.
- [ ] **Subservice organisations are identified and the carve-out or inclusive method is chosen** — most reports carve out the cloud provider and rely on complementary controls, which must then be listed.
- [ ] **Complementary user entity controls are drafted early** — these are the things your customers must do, and writing them at the end tends to produce an unrealistic list.
- [ ] **The auditor is a licensed CPA firm and their independence is confirmed** — the same firm cannot both build your controls and attest to them.

## 2. Control narratives and the system description {#control-narratives-and-the-system-description}

- [ ] **Every criterion maps to at least one control with a stated owner and frequency** — the mapping matrix is the backbone of the examination and gaps in it surface immediately.
- [ ] **Each narrative describes the control as it operates, including the system it runs in and who performs it** — write from the operator's perspective rather than from the policy.
- [ ] **Automated controls are distinguished from manual ones** — automated controls can be tested with a smaller sample, which materially reduces evidence work.
- [ ] **The system description covers infrastructure, software, people, procedures, and data** — omitting the people and procedures sections is a common first-draft mistake.
- [ ] **Control descriptions avoid promises you cannot evidence every time** — a narrative saying all changes are peer reviewed will be tested against every change in the sample, including the emergency ones.
- [ ] **Changes to the environment during the window are reflected in the narratives** — a migration mid-window means the control operated in two forms, and both need describing.
- [ ] **Someone who does not operate the control has read the narrative and agrees it is accurate** — self-written, self-reviewed narratives are where exceptions hide.

## 3. Evidence collection and sampling {#evidence-collection-and-sampling}

- [ ] **Every control has a defined evidence artefact, source system, and retention location** — decide this before the window, because reconstructing evidence afterwards is often impossible.
- [ ] **Evidence carries a reliable timestamp and shows the full population, not a filtered view** — auditors select samples from populations, so a screenshot of one approved ticket proves nothing about the other four hundred.
- [ ] **Population completeness can be demonstrated** — be ready to show that the change list, the user list, or the ticket export is complete, since an unverifiable population invalidates the sample drawn from it.
- [ ] **Periodic controls have evidence for every occurrence in the window** — a quarterly review missing one quarter is an exception, and there is no partial credit.
- [ ] **Evidence is stored where it survives staff departures and tool migrations** — screenshots in a personal drive or a chat thread will not be retrievable at fieldwork.
- [ ] **Automated evidence collection is preferred and its correctness has been verified** — compliance automation platforms mis-map controls often enough that spot-checking is mandatory.
- [ ] **Timeliness is evidenced, not just completion** — a review that was due in January and performed in April is an exception even though the artefact exists.

{{< alert context="danger" text="**Blocking:** a control that cannot produce evidence for every occurrence within the window will become an exception in the report. Identify these before the window opens and either fix the evidence path or remove the control from the description." />}}

## 4. Managing the observation window {#managing-the-observation-window}

- [ ] **A start date is fixed and every control is operating on day one** — not designed, not planned, operating.
- [ ] **A monthly internal check samples the same evidence the auditor will request** — catching a broken control in month two costs a fix; catching it at fieldwork costs a qualified report.
- [ ] **Exceptions found internally are logged with remediation and, where possible, a compensating record** — a documented, self-detected and corrected exception reads far better than one the auditor found.
- [ ] **Control owners are told what happens if they miss a cycle** — most window failures are a missed quarterly review by someone who did not know it mattered.
- [ ] **Personnel changes during the window are handled with a handover of control ownership** — an orphaned control quietly stops operating.
- [ ] **Tooling changes are planned around the window where possible** — migrating your ticketing system mid-window doubles the evidence work and often breaks population completeness.

## 5. Logical access and access reviews {#logical-access-and-access-reviews}

- [ ] **User access provisioning is tied to an approved request with evidence for every account in the sample** — including service accounts and accounts created during incidents.
- [ ] **Termination access removal is evidenced within the SLA the narrative claims** — compare the HR termination date against the deprovisioning timestamp for every leaver in the window, not just the sampled ones.
- [ ] **Periodic access reviews cover every in-scope system including the cloud console, database, code repository, and CI system** — reviewing only the identity provider misses the systems with direct local accounts.
- [ ] **Access review evidence shows the reviewer, the date, the full user list, and the actions taken** — a review with no revocations across a year of staff churn is not credible.
- [ ] **Privileged and administrative access is separately identified and justified** — with a break-glass procedure whose use is logged and reviewed.
- [ ] **Multi-factor authentication coverage is evidenced from configuration, not policy** — export the enrolment report and account for every exception.
- [ ] **Shared and generic accounts are eliminated or individually justified and monitored** — these break every attribution argument the report makes.

## 6. Change management evidence {#change-management-evidence}

- [ ] **Every production change traces to a ticket, a peer-reviewed pull request, and an approval before deployment** — the auditor will pull deployments from the pipeline log and work backwards.
- [ ] **The deployment log is the population, not the ticket queue** — if code can reach production without a ticket, the population is the deployment record and any gap is an exception.
- [ ] **Branch protection and required review are enforced technically, not by convention** — a repository setting is testable evidence; a team agreement is not.
- [ ] **Emergency change procedure exists and its retrospective approval is evidenced** — emergencies are expected, undocumented emergencies are not.
- [ ] **Segregation of duties between developer and deployer is evidenced or a compensating control is described** — small teams should describe the compensating review rather than claim a separation they do not have.
- [ ] **Infrastructure changes follow the same control as application changes** — infrastructure-as-code changes and manual console changes both need covering, and the console path usually does not.
- [ ] **Testing evidence exists for changes in the sample** — automated test results attached to the pipeline run are the cheapest form of this.

## 7. Vendor and subservice organisation management {#vendor-and-subservice-organisation-management}

- [ ] **A vendor inventory exists with a risk rating and a named business owner per vendor** — this is the population the vendor controls are tested against.
- [ ] **SOC 2 reports or equivalent assurance are obtained annually for critical vendors and actually read** — with the review recorded, including any exceptions noted in their report.
- [ ] **Complementary user entity controls listed in vendor reports are mapped to your own controls** — this is the step almost everyone skips and auditors increasingly test.
- [ ] **Vendor onboarding includes a security review before the contract is signed** — evidence of the review must predate the go-live date.
- [ ] **Bridge letters cover any gap between a vendor report period and your window** — request them early, since they are issued on the vendor's schedule.
- [ ] **Vendor offboarding evidences data return or destruction and access revocation** — including the API keys held in your own systems.

## 8. Readiness assessment and audit logistics {#readiness-assessment-and-audit-logistics}

- [ ] **A gap assessment has been run against the full criteria set before the window opens** — by someone independent of the people who built the controls.
- [ ] **Every gap has a remediation owner and a date that lands before the window start** — gaps closing during the window still produce exceptions for the earlier period.
- [ ] **Risk assessment, incident response, and business continuity have been performed within the last twelve months with evidence** — these are criteria in their own right and are frequently forgotten because they are not technical controls.
- [ ] **Policies are approved, dated, and acknowledged by staff with completion records** — the auditor samples the acknowledgement list against the employee roster.
- [ ] **Background checks, onboarding, and security training records exist for the hires in the window** — HR evidence is a common exception source because the records live outside the compliance team's tooling.
- [ ] **A single point of contact coordinates the auditor's requests and tracks the request list to closure** — scattered responses across many people slow fieldwork and produce inconsistent answers.
- [ ] **A dry run of the evidence request list has been completed** — pull ten sample requests and time how long each takes to satisfy.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Scope and trust services criteria selection | | | Pass / Pass with actions / Fail |
| Control narratives and the system description | | | Pass / Pass with actions / Fail |
| Evidence collection and sampling | | | Pass / Pass with actions / Fail |
| Managing the observation window | | | Pass / Pass with actions / Fail |
| Logical access and access reviews | | | Pass / Pass with actions / Fail |
| Change management evidence | | | Pass / Pass with actions / Fail |
| Vendor and subservice organisation management | | | Pass / Pass with actions / Fail |
| Readiness assessment and audit logistics | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated remediation item and confirm it is closed before the observation window opens, since remediation during the window still produces exceptions.

## Related checklists

- [ISO/IEC 27001 ISMS](/docs/compliance/iso27001-isms/)
- [Vendor Security Assessment](/docs/compliance/vendor-security-assessment/)
- [Secrets Management](/docs/security/secrets-management/)
- [CI/CD Pipeline](/docs/devops/cicd-pipeline/)
- [Employee IT Offboarding](/docs/itsm/employee-it-offboarding/)

## References

- [AICPA and CIMA — SOC suite of services](https://www.aicpa-cima.com/resources/landing/system-and-organization-controls-soc-suite-of-services)
- [AICPA and CIMA](https://www.aicpa-cima.com/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [CIS Critical Security Controls](https://www.cisecurity.org/controls)
