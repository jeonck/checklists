---
title: "Postmortem"
description: "Verify an incident review is blameless, accurate, and produces action items that actually get done."
icon: "history_edu"
weight: 540
toc: true
tags: ["postmortem", "incident-response", "sre", "continuous-improvement"]
---

A postmortem is not a report you file to prove the incident was handled. It is the mechanism by which an organisation converts one expensive outage into knowledge that prevents the next several. This checklist covers the writing, the analysis, the action items, and the review meeting — in the order the document is produced.

{{< alert context="info" text="**Who runs this:** the postmortem owner named at the end of the incident, with the incident commander and responders as contributors. **When:** draft within five working days of resolution, reviewed within ten." />}}

## 1. Scope, ownership, and timing

- [ ] **A single named owner is responsible for the document** — shared ownership reliably produces no document at all.
- [ ] **The trigger threshold for writing a postmortem is written policy** — typically any customer-visible incident above a defined severity, any data loss, and any incident that recurred.
- [ ] **The draft is written within five working days** — recall degrades quickly and responders move on to other work.
- [ ] **Contributors are given time to write it as part of their normal workload** — a postmortem written entirely in someone's evenings is a postmortem that gets skipped next time.
- [ ] **A near miss can be written up voluntarily** — the cheapest lessons come from incidents that were caught before customers noticed, and these only surface if writing them is welcomed.
- [ ] **The document lives in a searchable shared archive** — the value compounds only if a future engineer can find the one from eighteen months ago.

## 2. Blameless writing

- [ ] **The document describes systems and decisions, not people** — name roles or teams rather than individuals, since the goal is a system that tolerates ordinary human error.
- [ ] **Every human action is framed with the information available at the time** — someone acted reasonably given what their dashboard showed, and the fix belongs to the dashboard.
- [ ] **Counterfactuals are removed from the analysis** — sentences beginning "if only they had" describe a world that did not exist and produce no actionable change.
- [ ] **The words "human error" and "operator mistake" do not appear as conclusions** — they are the point at which analysis stops rather than the point at which it should start.
- [ ] **Hindsight-loaded language is edited out** — "obviously", "clearly", and "should have known" all signal that the writer is judging rather than explaining.
- [ ] **The review confirms that responders would be comfortable if the document were read widely** — if people would hesitate to be named, the culture is not yet blameless and the next incident will be reported late or not at all.

{{< alert context="danger" text="**This is the load-bearing item on the page.** The moment a postmortem is used in a performance conversation, engineers start hiding incidents, and every subsequent postmortem becomes fiction. Blamelessness is not politeness; it is what makes the data trustworthy." />}}

## 3. Timeline construction

- [ ] **The timeline is built from sources, not from memory** — chat transcripts, alert history, deploy logs, and the scribe's notes, each entry linked to its evidence.
- [ ] **Every entry carries an absolute timestamp with a stated timezone** — mixed local times across a distributed team make the sequence unreconstructable.
- [ ] **The timeline starts before the incident** — include the change, the configuration edit, or the traffic shift that set up the failure, which is often days earlier.
- [ ] **Key markers are called out explicitly** — first customer impact, first alert, first human acknowledgement, mitigation applied, and full recovery.
- [ ] **What responders believed at each point is recorded alongside what was actually true** — the gap between the two is usually the richest source of findings.
- [ ] **Detection delay and mitigation delay are stated as numbers** — these are the two levers you can actually pull, and they need to be measurable to be improved.
- [ ] **Actions that had no effect are included** — the dead ends explain where diagnosis time went and often point at missing signals.

## 4. Impact

- [ ] **Customer impact is quantified, not described** — affected users or requests, duration, and which functionality was degraded versus fully unavailable.
- [ ] **Error budget consumed is reported against the relevant SLO** — this connects the incident to the reliability decisions the team has already agreed.
- [ ] **Data impact is stated explicitly** — records lost, duplicated, or corrupted, and whether reconciliation was completed or is still outstanding.
- [ ] **Internal cost is recorded** — responder hours, work displaced, and the number of people who lost sleep, since this is what justifies the prevention work.
- [ ] **Financial and contractual consequences are noted where they apply** — SLA credits, lost transactions, and any regulatory notification that was triggered.

## 5. Contributing factors and root cause

- [ ] **Multiple contributing factors are identified, not a single root cause** — complex systems fail through combinations, and a single-cause narrative closes the analysis prematurely.
- [ ] **Each contributing factor is categorised** — trigger, latent condition, missing safeguard, detection gap, or response impediment, since each category attracts a different kind of fix.
- [ ] **Latent conditions that existed long before the incident are named** — the missing timeout, the unbounded queue, or the untested failover was there for months and will still be there tomorrow.
- [ ] **The analysis asks why the safeguards did not work, not only why the failure happened** — every serious incident passed through defences that were supposed to stop it.
- [ ] **Detection and response are analysed separately from causation** — an incident that would have lasted two minutes with better alerting is primarily a detection problem.
- [ ] **Any recurrence of a previous incident is linked to it** — and the action items from that earlier postmortem are checked for completion, which is usually where the answer is.
- [ ] **Speculation is labelled as speculation** — where the cause could not be conclusively determined, say so rather than committing to a plausible guess that ends the investigation.

## 6. Action items

- [ ] **Every action item has one named individual owner and a due date** — a team name is not an owner and "next quarter" is not a date.
- [ ] **Action items are tracked in the normal backlog, not only inside the document** — work that lives in a wiki page is work that does not get scheduled.
- [ ] **Each item is classified by whether it prevents, detects, or mitigates** — a set of items that are all detection improvements means nothing is actually being fixed.
- [ ] **Items are specific enough to be verifiably complete** — "improve monitoring" cannot be closed honestly, whereas "add a burn-rate alert on the checkout SLO" can.
- [ ] **The list is prioritised and deliberately short** — twenty low-priority items produce nothing; three completed high-priority items prevent the recurrence.
- [ ] **Items intentionally not being done are recorded as accepted risk with a decision-maker** — an explicit acceptance is far better than an item that quietly ages out.
- [ ] **Reverting temporary mitigations from the incident appears as an action item** — the emergency capacity increase and the disabled feature flag both need an owner.
- [ ] **Overdue action items are escalated on a schedule** — a monthly review of open postmortem actions catches the drift before the incident repeats.

## 7. The review meeting

- [ ] **The draft is circulated for reading before the meeting** — reading the document aloud consumes the whole session and prevents discussion.
- [ ] **Everyone who responded is invited, plus someone from outside the team** — an outsider asks the questions insiders have stopped noticing.
- [ ] **The meeting focuses on the analysis and the action items, not on re-narrating the timeline** — the timeline should be settled in the draft, with corrections collected in writing.
- [ ] **Disagreements about contributing factors are recorded in the document rather than resolved by seniority** — a documented disagreement is more useful than a false consensus.
- [ ] **Action items are agreed with their owners present and consenting** — an owner assigned in absentia has not accepted anything.
- [ ] **The meeting is time-boxed and ends with an explicit decision to publish** — postmortems that stay in draft indefinitely deliver none of their value.

## 8. Publication and organisational learning

- [ ] **The final document is published to the whole engineering organisation** — the audience that learns most is the teams who were not involved.
- [ ] **A short summary is written for non-engineering stakeholders where impact warrants it** — support, sales, and leadership need the impact and the prevention plan, not the stack traces.
- [ ] **Findings that apply to other services are routed to those teams explicitly** — the same missing timeout usually exists in four other places.
- [ ] **Runbooks and alerts are updated as part of closing the postmortem** — the knowledge is only durable once it lives where the next responder will look.
- [ ] **Postmortems are reviewed in aggregate at least quarterly** — themes across ten incidents reveal systemic problems that no single review can see.
- [ ] **Action item completion rate is tracked as a team metric** — a low completion rate is the single best predictor that the same incident will happen again.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Scope, ownership, and timing | | | Pass / Pass with actions / Fail |
| Blameless writing | | | Pass / Pass with actions / Fail |
| Timeline construction | | | Pass / Pass with actions / Fail |
| Impact | | | Pass / Pass with actions / Fail |
| Contributing factors and root cause | | | Pass / Pass with actions / Fail |
| Action items | | | Pass / Pass with actions / Fail |
| The review meeting | | | Pass / Pass with actions / Fail |
| Publication and organisational learning | | | Pass / Pass with actions / Fail |

Do not publish the postmortem until every row reads Pass or has a dated ticket attached to its outstanding actions.

## Related checklists

- [Incident Management](/docs/operations/incident-management/)
- [On-Call Handover](/docs/operations/on-call-handover/)
- [Observability](/docs/operations/observability/)
- [Security Incident Response](/docs/security/incident-response/)
- [Disaster Recovery Drill](/docs/itsm/disaster-recovery-drill/)

## References

- [Google SRE Book — Postmortem Culture: Learning from Failure](https://sre.google/sre-book/postmortem-culture/)
- [Google SRE Workbook — Postmortem Culture: Putting Blameless into Practice](https://sre.google/workbook/postmortem-culture/)
- [PagerDuty Incident Response — Postmortems](https://response.pagerduty.com/after/post_mortem_process/)
- [Atlassian Incident Management Handbook](https://www.atlassian.com/incident-management)
