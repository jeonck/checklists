---
title: "Change Management"
description: "Verify a change is classified, assessed, approved, scheduled, and reversible before it touches production."
icon: "change_circle"
weight: 930
toc: true
tags: ["itsm", "change-management", "release", "governance"]
---

Change management exists to stop two failure modes: the risky change that nobody reviewed, and the review process so heavy that people route around it. The balance comes from classification — most changes should be pre-approved standard changes that flow through automation, leaving human attention for the genuinely novel ones. Use this checklist for a normal change heading to a change advisory board, and use the emergency section when something is already on fire.

{{< alert context="info" text="**Who runs this:** the change owner, reviewed by a peer and authorised by the change authority for the affected service. **When:** before the change is submitted for approval, and again at the post-implementation review." />}}

## 1. Change classification {#change-classification}

- [ ] **The change is classified as standard, normal, or emergency, and the classification is justified** — misclassifying a normal change as standard to skip approval is the most common abuse of the process.
- [ ] **Standard changes come from a pre-approved catalogue with a documented procedure and a known risk profile** — if the procedure is not written down, it is not a standard change no matter how often you do it.
- [ ] **Repeated normal changes are reviewed as candidates for the standard catalogue** — a process that keeps sending routine work to a board trains people to bypass the board.
- [ ] **The change record names the affected services and configuration items** — impact analysis is impossible if the record says infrastructure update and nothing else.
- [ ] **The change has a single accountable owner, not a team alias** — an alias in the owner field means nobody answers when the rollout misbehaves at 02:00.
- [ ] **The desired outcome is stated in business terms, not just technical ones** — this is what the approver is actually approving.

## 2. Risk and impact assessment {#risk-and-impact-assessment}

- [ ] **The blast radius is stated: which users, which services, and which data are affected if this goes wrong** — including the services that merely depend on the one being changed.
- [ ] **The change is assessed against the worst realistic outcome, not the expected one** — likelihood and impact scored separately, so a low-probability catastrophic change is not averaged into looking routine.
- [ ] **Downstream and upstream dependencies have been consulted, not merely listed** — the team that consumes your API has an opinion about your schema change.
- [ ] **Data-affecting changes are identified explicitly** — anything that migrates, transforms, or deletes data is a different risk class from a configuration change, because rollback does not restore data.
- [ ] **Security and compliance impact is assessed where the change touches authentication, network boundaries, encryption, or personal data** — these need the security owner's input before the board, not after.
- [ ] **Conflicting or overlapping changes in the same window are identified from the change calendar** — two safe changes in the same window can combine into an unsafe one.
- [ ] **The cost of not doing the change is stated** — an expiring certificate or an unpatched critical vulnerability makes deferral the risky option.

## 3. Approval and authorisation {#approval-and-authorisation}

- [ ] **The approver is the change authority for the affected service and is not the person implementing the change** — self-approval defeats the entire control.
- [ ] **Peer technical review has happened and the reviewer is named in the record** — a review nobody signed is a review nobody did.
- [ ] **The approval covers the specific scope described, and any scope expansion returns for re-approval** — while I was in there is how unrelated outages start.
- [ ] **Business sign-off exists where the change affects customer-visible behaviour or contractual commitments** — technical approval does not cover a change that breaks a customer integration.
- [ ] **Approvals are recorded with timestamp and identity in the change system, not in chat** — chat approvals cannot be produced at audit and cannot be searched at incident review.
- [ ] **The change is rejected or deferred if any required approval is missing at the cut-off** — a process that lets changes proceed pending approval has no approval step.

## 4. Scheduling and freeze windows {#scheduling-and-freeze-windows}

- [ ] **The window is chosen against actual traffic data, not by assumption about quiet periods** — quiet for one region is peak for another.
- [ ] **The window includes time for verification and for rollback, not just for implementation** — a 30-minute window for a change with a 45-minute rollback is not a plan.
- [ ] **The change does not fall inside a declared freeze period, or has a documented freeze exception** — freeze exceptions need the same rigour as emergency changes.
- [ ] **The change calendar has been checked for collisions with other teams and with vendor maintenance** — including the cloud provider and network carrier maintenance notices you subscribed to and stopped reading.
- [ ] **Sufficient staff are available and awake for the duration, including the people needed for rollback** — a change implemented by the only person who understands it, alone, at 03:00, is a single point of failure.
- [ ] **On-call is briefed that the change is happening and knows how to identify symptoms caused by it** — the first responder to the resulting alert is frequently not the implementer.
- [ ] **Business events that would amplify an outage are checked** — month-end close, a marketing launch, a regulatory filing deadline.

## 5. Implementation plan {#implementation-plan}

- [ ] **The plan is written as ordered, executable steps that a competent colleague could follow** — if only the author can run it, the author is also the only possible rollback operator.
- [ ] **Every step states the expected result so deviation is detectable immediately** — a plan without expected outputs cannot tell you when to stop.
- [ ] **The change has been rehearsed in a non-production environment that resembles production in the ways that matter** — same version, same data shape, same scale characteristics for the parts under change.
- [ ] **Pre-change state is captured: configuration, versions, metrics baseline, and a fresh backup where data is involved** — you cannot prove you restored something if you did not record what it looked like.
- [ ] **Automated changes are executed from version-controlled code, not from a console session** — a console change is unreviewable and unrepeatable.
- [ ] **Manual steps are minimised and the remaining ones are explicitly flagged as the risky part** — most change-induced incidents trace to a hand-typed step.
- [ ] **Go and no-go criteria are defined before the window opens** — deciding whether the results look good enough while under time pressure produces optimistic answers.

## 6. Back-out plan {#back-out-plan}

- [ ] **A back-out plan exists, is specific, and has been tested** — restore from backup is not a back-out plan, it is an aspiration with an unknown duration.
- [ ] **The time to execute the back-out is measured and fits inside the window** — this figure determines the latest safe moment to abort.
- [ ] **The point of no return is identified and stated in the plan** — the step after which rollback stops being possible, usually a data migration or an external notification.
- [ ] **Data written by the new version is accounted for in the rollback** — if the previous version cannot read it, rolling back the code without the data is a corruption event.
- [ ] **The abort decision has a named decision-maker and a trigger condition** — nobody rolls back voluntarily at minute 55 of a 60-minute window without a pre-agreed rule.
- [ ] **The back-out works even if the deployment tooling is the thing that failed** — a rollback that depends on the broken pipeline is not available when you need it.

{{< alert context="warning" text="**Blocking:** a change that alters or migrates data without a tested restore path should not be approved. Code rollback is cheap; data rollback is frequently impossible, and discovering that during the change window is how a one-hour maintenance becomes a multi-day recovery." />}}

## 7. Communication {#communication}

- [ ] **Affected users are notified in advance with the window, the expected impact, and where to report problems** — including the internal teams whose work depends on the service.
- [ ] **The status page or equivalent is updated before the window opens, not after somebody complains** — the support queue is a slower and more expensive notification channel.
- [ ] **A communication channel for the change is open for the duration with all participants present** — retrospectives are far easier when the timeline is already written down in one place.
- [ ] **Customers and external parties with contractual notice periods are informed within those periods** — some integration partners have a contractual right to advance notice of interface changes.
- [ ] **Completion or abort is communicated on the same channels as the announcement** — silence after a maintenance window is interpreted as failure.

## 8. Execution and verification {#execution-and-verification}

- [ ] **The implementer follows the approved plan and records deviations as they happen** — reconstructing what actually happened from memory afterwards produces a tidy fiction.
- [ ] **Verification tests the business function, not just that the deployment finished** — a green pipeline says the artefact shipped, not that customers can check out.
- [ ] **Monitoring and error rates are watched for a defined soak period before the change is declared successful** — many change-induced failures appear at the next traffic peak, not at minute two.
- [ ] **A rollback triggered mid-change is itself verified against the pre-change baseline** — a failed rollback is more dangerous than the original change.
- [ ] **The change record is updated with the actual start time, end time, and outcome** — these are the numbers that make change failure rate meaningful later.
- [ ] **Any temporary measure applied during the window is removed or ticketed** — disabled alerts, relaxed rate limits, and paused jobs get forgotten with impressive reliability.

## 9. Post-implementation review and emergency change retrospective {#post-implementation-review-and-emergency-change-retrospective}

- [ ] **Every failed or backed-out change gets a review focused on the process, not the person** — the useful output is which control would have caught this, not who typed the command.
- [ ] **Change failure rate and mean time to restore are tracked over time and reviewed as trends** — a rising failure rate usually means the approval step has become a rubber stamp.
- [ ] **Every emergency change is retrospectively reviewed within a defined period, typically five working days** — the emergency path exists for speed, and its only safeguard is that the review actually happens.
- [ ] **Emergency changes are checked for whether they were genuinely emergencies** — if a third of changes are emergencies, the normal path is too slow and people are gaming it.
- [ ] **Documentation, configuration management records, and runbooks are updated to reflect the new state** — an out-of-date configuration record makes the next impact assessment wrong.
- [ ] **Actions from the review have owners and dates and are tracked to closure** — an action list without dates is a record of good intentions.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Change classification | | | Pass / Pass with actions / Fail |
| Risk and impact assessment | | | Pass / Pass with actions / Fail |
| Approval and authorisation | | | Pass / Pass with actions / Fail |
| Scheduling and freeze windows | | | Pass / Pass with actions / Fail |
| Implementation plan | | | Pass / Pass with actions / Fail |
| Back-out plan | | | Pass / Pass with actions / Fail |
| Communication | | | Pass / Pass with actions / Fail |
| Execution and verification | | | Pass / Pass with actions / Fail |
| Post-implementation review | | | Pass / Pass with actions / Fail |

Attach this completed checklist to the change record, and treat any Fail in classification, approval, or back-out as a blocker rather than a conditional pass.

## Related checklists

- [Release Day](/docs/devops/release-day/)
- [Network Change](/docs/networking/network-change/)
- [Database Schema Migration](/docs/development/database-schema-migration/)
- [Incident Management](/docs/operations/incident-management/)
- [Production Readiness Review](/docs/devops/production-readiness/)

## References

- [ITIL Service Management (Axelos)](https://www.axelos.com/certifications/itil-service-management)
- [ISO/IEC 20000-1 — Service Management System Requirements](https://www.iso.org/standard/70636.html)
- [ISO/IEC 27001 — Information Security Management](https://www.iso.org/standard/27001)
- [Google SRE Workbook](https://sre.google/workbook/table-of-contents/)
- [AWS Well-Architected Framework — Operational Excellence Pillar](https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/welcome.html)
