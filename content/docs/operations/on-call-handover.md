---
title: "On-Call Handover"
description: "Verify nothing is dropped when the pager changes hands between two on-call shifts."
icon: "phone_in_talk"
weight: 520
toc: true
tags: ["on-call", "operations", "sre", "handover"]
---

Most on-call failures are not heroic technical failures. They are an ongoing issue that the outgoing engineer knew about, forgot to mention, and that paged the incoming engineer at 2am with no context. A handover is a short, structured conversation with a written artefact attached; this checklist covers what each side owes the other and what has to be true before the pager actually moves.

{{< alert context="info" text="**Who runs this:** the outgoing and incoming on-call engineers together, with the team lead as escalation if something cannot be handed over cleanly. **When:** at a fixed time at the end of every shift, before the rotation flips in the paging tool." />}}

## 1. Before the handover meeting

- [ ] **The handover happens at a scheduled time, not whenever the rotation silently flips** — an automatic rotation change with no conversation is how context is lost.
- [ ] **The outgoing engineer has written the handover note before the meeting starts** — the meeting is for questions and clarification, not for composing the note live.
- [ ] **The handover note lives in a persistent, searchable place** — a ticket, a channel with retention, or the runbook repository, not a direct message that nobody else can find.
- [ ] **Both engineers have the shift open in front of them** — the alert history, the incident tracker, and the change log for the period, so nothing is recalled from memory alone.
- [ ] **The meeting is time-boxed to fifteen minutes** — if it needs longer, that is a signal to escalate an unresolved issue rather than to talk for an hour.
- [ ] **A handover is still held when the shift was quiet** — the absence of pages is itself information, and skipping the ritual is how the ritual dies.

## 2. What the outgoing engineer hands over

- [ ] **Every page received during the shift, with what was done about it** — including pages that were acknowledged and ignored, which are the most likely to recur.
- [ ] **Open incidents with current status, severity, and who else is involved** — never hand over an active incident by pager alone; brief the incoming engineer directly.
- [ ] **Any temporary mitigation still in place** — a scaled-up cluster, a disabled feature flag, a paused consumer, or a manually restarted job, each with the condition for reverting it.
- [ ] **Silenced or suppressed alerts, with the expiry time for each silence** — a permanent silence created at 3am is a future outage nobody will see coming.
- [ ] **Anything that looked odd but did not page** — an unexplained latency step, a growing queue, a new error signature; these are the early warning that the next shift will need.
- [ ] **Manual work in flight** — a half-finished migration, a partially applied configuration change, or a rollback that was started and not finished.
- [ ] **Tickets raised during the shift, with links** — so follow-up work does not have to be reconstructed from the chat history.

## 3. What the incoming engineer confirms

- [ ] **The pager actually reaches them** — send a test notification and confirm it arrives on the phone that will be in the room, with the ringer on and do-not-disturb bypassed.
- [ ] **Access works before it is needed** — production console, cluster credentials, VPN, the incident tool, and any break-glass procedure, all exercised now rather than mid-incident.
- [ ] **They can reach the escalation path** — the secondary on-call, the incident commander pool, and the vendor support contact, with numbers that have been dialled at least once.
- [ ] **They have read the handover note and asked their questions** — the handover is complete when the incoming engineer can restate the open items, not when the outgoing engineer stops talking.
- [ ] **They know which changes are planned during their shift** — releases, migrations, certificate rotations, and third-party maintenance windows all raise the probability of a page.
- [ ] **They confirm their own availability constraints** — travel, poor signal, or personal commitments must be flagged now so cover can be arranged, not discovered when a page goes unanswered.

{{< alert context="danger" text="**Blocking:** do not flip the rotation until the incoming engineer has confirmed a test page arrived and that production access works. A silent pager is indistinguishable from a healthy system until the outage is already an hour old." />}}

## 4. System state review

- [ ] **The service dashboards are walked through together** — current error rate and latency versus the same time last week, so a slow drift is visible rather than normalised.
- [ ] **Error budget consumption for the shift is reported** — a large burn during a quiet shift means something is degrading without paging.
- [ ] **Capacity headroom is checked** — disk, database connections, queue depth, and any resource that is consumed monotonically and cannot be recovered without action.
- [ ] **Certificate and credential expiries within the next fourteen days are listed** — expiry-driven outages are entirely predictable and entirely avoidable.
- [ ] **Backlogs and dead letter queues are reported with their trend** — a dead letter queue that grew during the shift is a customer-impacting bug that has not surfaced yet.
- [ ] **Known-degraded dependencies are named** — including third-party providers currently on a status-page incident, so the next page is not misdiagnosed.

## 5. Alerting and paging hygiene

- [ ] **Alerts that fired more than twice this shift are flagged as noisy** — repeated pages for the same non-actionable condition are the main driver of on-call attrition.
- [ ] **Every silence created during the shift has an owner and an expiry** — and silences that are about to expire are transferred explicitly.
- [ ] **New alerts added during the shift are reviewed for a runbook link** — an alert authored at 3am usually has no documented response yet.
- [ ] **False positives are ticketed, not just tolerated** — if it is not written down, the same alert will wake five more people before anyone fixes it.
- [ ] **The paging tool's schedule shows the correct people for the next fourteen days** — including holiday cover, and with an override rather than a private arrangement.

## 6. Documentation and runbooks

- [ ] **Any runbook used during the shift was corrected where it was wrong** — the moment you discover a stale step is the only moment you will reliably remember to fix it.
- [ ] **Any incident resolved without a runbook produced one** — even a rough set of commands is better than the next responder starting from nothing.
- [ ] **New or changed escalation contacts are recorded in the shared directory** — not in one engineer's phone.
- [ ] **Links in the handover note resolve** — dashboards, tickets, and runbooks should be checked, since a dead link during an incident costs minutes you do not have.
- [ ] **Manual steps performed during the shift are captured as automation candidates** — the third time a human runs the same recovery command, it should become a script.
- [ ] **The runbook index reflects any service added or retired since the last shift** — responders will search the index first, and a missing entry reads as no procedure exists.

## 7. Escalation and follow-up ownership

- [ ] **Each open item has exactly one named owner after the handover** — either the incoming engineer or a specific person on the team, never "on-call" as an abstraction.
- [ ] **Items that need daytime engineering work are moved into the team backlog, not carried in the pager** — on-call is for response; recurring toil belongs in sprint planning.
- [ ] **Anything unresolved for more than two consecutive shifts is escalated to the team lead** — repeated handover of the same problem is the signal that on-call cannot fix it alone.
- [ ] **Incidents from the shift have a postmortem owner assigned where the severity threshold was met** — postmortems left unassigned at handover are rarely written.
- [ ] **Follow-up actions from previous postmortems that are now overdue are surfaced** — the same incident recurring is almost always an unimplemented action item.

## 8. Shift health and sustainability

- [ ] **Overnight pages are recorded with their timestamps** — sleep interruption is the metric that predicts burnout, and it is invisible unless it is counted.
- [ ] **The outgoing engineer takes compensating rest after a disrupted night** — agreed as team policy, so that taking it does not require asking permission.
- [ ] **The number of actionable versus non-actionable pages is tracked per shift** — the ratio, not the raw count, tells you whether alerting is working.
- [ ] **On-call load is reviewed at least monthly against a target** — a commonly used ceiling is around two pages per shift; sustained breaches justify pausing feature work.
- [ ] **No single person holds the pager for consecutive rotations without agreement** — informal cover arrangements hide load from everyone who could fix it.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Before the handover meeting | | | Pass / Pass with actions / Fail |
| What the outgoing engineer hands over | | | Pass / Pass with actions / Fail |
| What the incoming engineer confirms | | | Pass / Pass with actions / Fail |
| System state review | | | Pass / Pass with actions / Fail |
| Alerting and paging hygiene | | | Pass / Pass with actions / Fail |
| Documentation and runbooks | | | Pass / Pass with actions / Fail |
| Escalation and follow-up ownership | | | Pass / Pass with actions / Fail |
| Shift health and sustainability | | | Pass / Pass with actions / Fail |

Attach the completed table to the handover note and raise a dated ticket for every item marked "Pass with actions" before the rotation flips.

## Related checklists

- [Incident Management](/docs/operations/incident-management/)
- [Observability](/docs/operations/observability/)
- [Postmortem](/docs/operations/postmortem/)
- [Production Readiness Review](/docs/devops/production-readiness/)
- [Release Day](/docs/devops/release-day/)

## References

- [Google SRE Book — Being On-Call](https://sre.google/sre-book/being-on-call/)
- [Google SRE Workbook — On-Call](https://sre.google/workbook/on-call/)
- [PagerDuty Incident Response Documentation](https://response.pagerduty.com/)
- [Google SRE Book — Practical Alerting](https://sre.google/sre-book/practical-alerting/)
