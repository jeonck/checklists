---
title: "Disaster Recovery Drill"
description: "Verify a recovery exercise is scoped, safe, and measured against the stated RTO and RPO targets."
icon: "crisis_alert"
weight: 940
toc: true
tags: ["itsm", "disaster-recovery", "resilience", "business-continuity"]
---

A disaster recovery plan that has never been executed is a document, not a capability. The point of a drill is to convert assumptions into measurements: how long recovery actually takes, how much data is actually lost, and which step in the runbook turns out to reference a server that was decommissioned two years ago. Run this in time order — choose the scenario, bound the blast radius, tell the right people, execute, measure, and write it up — and resist the urge to pick a scenario you already know you will pass.

{{< alert context="info" text="**Who runs this:** a drill lead outside the team being tested, with the service owner, an observer taking timings, and a safety officer empowered to stop the exercise. **When:** at least annually per critical service, and after any material change to architecture, dependencies, or recovery tooling." />}}

## 1. Objectives and scenario selection

- [ ] **The drill has a stated hypothesis to test, not just an activity to perform** — for example, we can restore the order database to a working state within four hours with at most 15 minutes of data loss.
- [ ] **The scenario is chosen from a risk assessment rather than from convenience** — the scenarios you avoid rehearsing are precisely the ones you are least able to survive.
- [ ] **The scenario is realistic for this organisation** — region loss, ransomware encryption of shared storage, loss of the identity provider, or a critical SaaS vendor outage all beat a generic server failed.
- [ ] **Recovery of the identity provider is exercised at some point in the drill programme** — if authentication is down, the team may be unable to log into the tools they need to recover anything else.
- [ ] **The exercise type is chosen deliberately: tabletop, partial failover, or full failover** — a tabletop finds gaps in decision-making, only a live failover finds gaps in the tooling.
- [ ] **The success criteria are written down before the drill starts** — criteria agreed afterwards are always met.
- [ ] **Previous drill findings are on the agenda so repeat failures are visible** — the same finding appearing three years running is itself the most important result.

## 2. Scope and blast radius

- [ ] **The systems in scope and explicitly out of scope are listed** — ambiguity here is what turns a drill into an incident.
- [ ] **The drill runs in an environment where the impact is understood and accepted** — a production failover is the most valuable and the most dangerous option, and it needs explicit business acceptance.
- [ ] **Shared dependencies that the drill could disturb are identified** — a shared database, a shared network path, or a shared identity tenant means the blast radius is larger than the named systems.
- [ ] **Customer-facing impact is quantified in advance and approved by a business owner** — including whether any real transactions could be lost.
- [ ] **The failback path is planned before the failover starts** — returning to normal is often harder than failing over, and teams routinely run out of window during failback.
- [ ] **Data written during the drill is prevented from polluting production systems** — test orders, test notifications, and test payments reaching real customers or real ledgers is a classic drill-induced incident.
- [ ] **A hard stop time is agreed after which the exercise is abandoned regardless of progress** — without it, drills expand until they become the outage they were simulating.

## 3. Planning, participants, and notification

- [ ] **The participants are the people who would actually respond, not a hand-picked expert team** — a drill run by the architect who built the system measures the architect, not the organisation.
- [ ] **At least one participant is deliberately unfamiliar with the system** — they will find the runbook steps that only make sense to the author.
- [ ] **Roles are assigned in advance: incident lead, communications, technical operators, observer, and safety officer** — with the observer explicitly not helping, so the timings stay honest.
- [ ] **A person with authority to stop the exercise is named and their word is final** — and everyone knows the phrase that means this is real, stop the drill.
- [ ] **Stakeholders are notified: on-call, the service desk, security operations, and anyone monitoring the systems involved** — an unannounced failover generates a real incident response, wasting the responders' night.
- [ ] **External parties are notified where their systems or alerts will see the effects** — cloud provider support, monitoring vendors, and any partner whose integration will fail.
- [ ] **The drill is announced on the change calendar as a change, with the usual approvals** — a drill is a change to production and deserves the same governance.
- [ ] **A separate communication channel is set up so drill traffic is unmistakably distinguishable from real incident traffic** — every message is prefixed as an exercise.

{{< alert context="warning" text="**Blocking:** never start a live failover without a named person who can call a stop, a pre-agreed phrase that distinguishes a real incident from the exercise, and a tested failback plan. Drills that turn into genuine outages almost always trace to one of these three being missing." />}}

## 4. Pre-drill readiness verification

- [ ] **The recovery runbook is at hand in a form that survives the scenario** — a runbook stored only in the wiki hosted in the region you are about to fail is not available.
- [ ] **Credentials needed for recovery are accessible without the systems being recovered** — the break-glass path, tested, not assumed.
- [ ] **Backups and replicas that the drill depends on are confirmed present and within their expected recency** — verify before the clock starts, so a missing backup is a finding rather than a wasted exercise.
- [ ] **Contact details for the people and vendors involved are current** — an out-of-date escalation number is one of the most commonly discovered findings.
- [ ] **Monitoring and logging for the recovery environment are working before the drill starts** — you cannot measure recovery in a system you cannot observe.
- [ ] **The timekeeping method is agreed and a single clock is used for all timestamps** — reconciling timings across time zones after the fact loses the precision that makes the measurement useful.

## 5. Running the drill

- [ ] **The scenario is injected as described and the start time is recorded to the minute** — the clock starts at detection or at injection, agreed in advance and applied consistently.
- [ ] **Participants follow the documented runbook rather than their own knowledge** — the point is to test the document; if they deviate, that deviation is the finding.
- [ ] **The observer records every step with a timestamp, including the waiting and the confusion** — the long pauses are where the recoverable time actually goes.
- [ ] **Decision points and who made each decision are captured** — decision latency is usually a larger component of recovery time than technical execution.
- [ ] **Blockers are recorded and worked around rather than fixed silently** — a quiet fix by the person who knows the trick removes the finding from the record.
- [ ] **Communication to simulated stakeholders is exercised, not skipped** — status updates during a real event consume real responder time and need practising.
- [ ] **If the exercise is stopped early, the reason and the state reached are recorded** — an aborted drill that documented why is more valuable than a completed one that fudged a step.
- [ ] **Failback is executed and timed as part of the drill, not deferred to whoever is around afterwards** — untimed failback is untested failback.

## 6. Measuring RTO and RPO against targets

- [ ] **Actual recovery time is measured from the agreed start point to verified service restoration** — restoration means the business function works, not that the instance is running.
- [ ] **Actual recovery point is measured as the real gap between the last recoverable transaction and the failure moment** — check the data, do not read the replication configuration and assume.
- [ ] **Measured values are compared against the documented targets and the gap is stated plainly** — a four-hour target met in eleven hours is a business risk decision, not a technical footnote.
- [ ] **Where targets were missed, the largest contributing time blocks are identified** — usually decision-making, access problems, or a manual data step, rather than the restore itself.
- [ ] **Targets that were met with unrealistic assistance are marked as not genuinely met** — an expert on the call who bypassed three runbook steps invalidates the measurement.
- [ ] **The measurement covers the full dependency chain, not just the primary system** — a recovered database with no working authentication, DNS, or network path has restored nothing.
- [ ] **Where the measured capability cannot meet the target, either the architecture or the target is changed** — carrying a target you have proven you cannot meet is worse than having no target.

## 7. Data integrity and recovery validation

- [ ] **Recovered data is validated against known-good reference values, not merely counted** — row counts match on datasets that are silently corrupt.
- [ ] **Referential consistency across systems recovered from different points in time is checked** — restoring a database and a message queue to different moments produces orphaned records.
- [ ] **Application-level functional tests are run against the recovered environment** — the critical user journey end to end, executed by someone who knows what correct looks like.
- [ ] **Backup integrity failures found during the drill are treated as incidents in their own right** — an unrestorable backup discovered in a drill is a production risk today, not a drill finding for the report.
- [ ] **Encryption keys, certificates, and secrets needed by the recovered systems are confirmed available in the recovery context** — a restored volume you cannot decrypt is not a restore.
- [ ] **Any data loss actually incurred during the exercise is quantified and reported** — including any real data affected if the drill touched production.

## 8. Findings, write-up, and follow-through

- [ ] **The write-up is produced within a defined period, typically five working days, while the detail is fresh** — a report written a month later contains the story and not the timings.
- [ ] **The report states measured RTO and RPO, the targets, and the delta, on the first page** — this is the number the business needs and the only one most readers will read.
- [ ] **Findings are recorded as tickets with owners and dates, not as a list in a document** — an unticketed finding will reappear in next year's drill.
- [ ] **Runbook corrections are applied immediately while the gaps are still remembered** — the highest-value hour of the whole exercise is the one straight afterwards.
- [ ] **The report is blameless and focuses on system and process gaps** — a drill that people fear looking bad in is a drill that gets quietly rehearsed in advance.
- [ ] **Risk acceptance is recorded and signed where a gap will not be fixed** — the business owner accepts the residual exposure explicitly rather than by default.
- [ ] **The date and scenario of the next drill are set before this one is closed** — including a rerun of anything that failed, sooner than the annual cycle.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Objectives and scenario selection | | | Pass / Pass with actions / Fail |
| Scope and blast radius | | | Pass / Pass with actions / Fail |
| Planning, participants, and notification | | | Pass / Pass with actions / Fail |
| Pre-drill readiness verification | | | Pass / Pass with actions / Fail |
| Running the drill | | | Pass / Pass with actions / Fail |
| RTO and RPO measurement | | | Pass / Pass with actions / Fail |
| Data integrity and recovery validation | | | Pass / Pass with actions / Fail |
| Findings, write-up, and follow-through | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with an owner, and escalate any missed recovery objective to the business owner of the affected service rather than closing it within IT.

## Related checklists

- [Backup and Recovery](/docs/operations/backup-and-recovery/)
- [Incident Management](/docs/operations/incident-management/)
- [Postmortem](/docs/operations/postmortem/)
- [Production Readiness Review](/docs/devops/production-readiness/)
- [Change Management](/docs/itsm/change-management/)

## References

- [NIST SP 800-34 Rev. 1 — Contingency Planning Guide for Federal Information Systems](https://csrc.nist.gov/pubs/sp/800/34/r1/upd1/final)
- [ISO 22301 — Business Continuity Management](https://www.iso.org/iso-22301-business-continuity.html)
- [AWS — Disaster Recovery of Workloads on AWS](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-workloads-on-aws.html)
- [AWS Well-Architected Framework — Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)
- [Google SRE Workbook](https://sre.google/workbook/table-of-contents/)
