---
title: "Incident Management"
description: "Verify an incident is detected, coordinated, communicated, and closed out without improvisation."
icon: "report"
weight: 530
toc: true
tags: ["incident-response", "operations", "sre", "on-call"]
---

An incident process exists so that nobody has to invent one under pressure. This checklist follows the shape of a real incident in order — detection, triage, roles, mitigation, communication, resolution, and the handover to a postmortem — and is meant to be worked through top to bottom while the incident is live, not read afterwards.

{{< alert context="info" text="**Who runs this:** the incident commander, supported by the ops lead and comms lead. **When:** from the moment an incident is declared until the postmortem owner has accepted the handover." />}}

## 1. Detection and declaration {#detection-and-declaration}

- [ ] **The incident has an explicit declaration, not a gradual realisation** — someone says the words and starts the clock, because an undeclared incident has no commander and no communications.
- [ ] **Anyone can declare an incident without asking permission** — the cost of a false declaration is a short call; the cost of a delayed one is measured in hours of customer impact.
- [ ] **Declaration creates the incident record automatically** — a ticket, a dedicated channel, and a bridge, so the responders spend their first minutes diagnosing rather than setting up tooling.
- [ ] **The detection source is recorded** — alert, synthetic probe, customer report, or internal observation; detection by customer report is itself a finding for the postmortem.
- [ ] **Time of first customer impact is estimated early and refined later** — it drives severity, SLA obligations, and any regulatory reporting clock.
- [ ] **Recent changes are pulled up immediately** — deploys, feature flag flips, configuration and infrastructure changes in the last few hours account for the large majority of incidents.

## 2. Triage and severity {#triage-and-severity}

- [ ] **Severity is assigned from a written matrix, not by feel** — the matrix should key on customer impact and scope, so that two different commanders reach the same answer.
- [ ] **Severity is set within the first few minutes and revised openly when it changes** — under-declaring to avoid waking people is the most expensive mistake in this process.
- [ ] **Blast radius is scoped explicitly** — which customers, which regions, which features, and whether the impact is total or partial degradation.
- [ ] **Data loss, data corruption, and security compromise are checked for by name** — each of these changes the response entirely and cannot be discovered late.
- [ ] **Each severity level maps to a defined response** — who is paged, how often updates go out, and whether executives and legal are notified.
- [ ] **Regulatory and contractual notification clocks are identified at triage** — breach notification and SLA credit windows start at impact, not at your convenience.

## 3. Roles {#roles}

- [ ] **An incident commander is named out loud and acknowledged by everyone on the call** — the commander coordinates and decides; they do not debug, because a commander with their head in a terminal is not commanding.
- [ ] **An ops lead owns the hands-on-keyboard work** — all changes to production during the incident go through this person, so that two responders never fight over the same system.
- [ ] **A comms lead owns all outward communication** — status page, customer messaging, and internal stakeholder updates, so responders are not interrupted to answer questions.
- [ ] **A scribe records a timestamped log of observations, decisions, and actions** — memory is unreliable and the postmortem timeline is impossible to reconstruct afterwards.
- [ ] **Roles are handed over explicitly when someone tires or leaves** — with a verbal summary and confirmation, never by simply going quiet.
- [ ] **Subject matter experts are pulled in on request and released when done** — an open call full of idle observers degrades signal for everyone remaining.
- [ ] **The commander role is separated from seniority** — the most senior person in the room is often the most useful debugging, and command is a distinct skill.

{{< alert context="warning" text="**Common failure:** nobody takes the commander role because everyone assumes someone else has it. If ten minutes pass with no named commander, the most recently paged responder takes it by default and says so in the channel." />}}

## 4. Mitigation {#mitigation}

- [ ] **Restoring service takes priority over understanding the cause** — the investigation continues in parallel, but mitigation is not blocked on a diagnosis.
- [ ] **Rollback is considered as the first option for any change-correlated incident** — reverting to a known-good state is faster and far less risky than fixing forward under pressure.
- [ ] **One change is made at a time, announced before and confirmed after** — simultaneous changes make it impossible to tell what worked and can compound the damage.
- [ ] **Every mitigating action is logged with its timestamp and its effect** — including actions that made no difference, which are just as important in the postmortem.
- [ ] **Emergency changes bypassing normal review are recorded for retrospective approval** — break-glass access should be usable and auditable, not blocked.
- [ ] **Temporary mitigations are ticketed for reversal at the moment they are applied** — an emergency capacity increase or disabled feature that nobody revisits becomes permanent cost or permanent breakage.
- [ ] **Evidence is preserved before destructive remediation** — capture logs, heap dumps, and a snapshot of the failing node before restarting or replacing it, or the cause is gone forever.

## 5. Communication cadence {#communication-cadence}

- [ ] **The status page is updated within the response window defined for the severity** — customers who cannot tell whether the problem is yours will flood support and assume the worst.
- [ ] **Updates go out on a fixed cadence even when there is nothing new** — an update saying the investigation continues and the next update is in thirty minutes prevents the escalation phone calls.
- [ ] **Customer-facing messages describe impact and workaround, not internal architecture** — say which features are affected and what a user can do, and avoid speculating about cause.
- [ ] **Internal and external communications are consistent** — leaked contradictions between an internal channel and a public status page cost more trust than the outage did.
- [ ] **Support and account teams get a briefing they can paste to customers** — otherwise each one improvises a different and probably wrong answer.
- [ ] **One channel is designated as the source of truth** — side conversations in direct messages fragment the record and hide decisions from the scribe.
- [ ] **Executives receive a summary on a separate track from the response channel** — so that stakeholder questions do not interrupt the responders.

## 6. Resolution {#resolution}

- [ ] **Recovery is verified from the customer's perspective before declaring resolution** — check the synthetic probe and a real user journey, not just that the error graph came down.
- [ ] **Backlogs, queues, and retry storms are drained and confirmed healthy** — a restored service can immediately fall over again under the queued load released at recovery.
- [ ] **Data written during the incident is checked for correctness** — partially processed transactions, duplicated messages, and skipped records need explicit reconciliation.
- [ ] **The all-clear is announced in every channel where the incident was announced** — including the status page, which is commonly left showing a resolved incident as ongoing.
- [ ] **Monitoring is watched for an agreed stabilisation period before standing down** — resolving too early and re-declaring twenty minutes later damages credibility more than waiting.
- [ ] **Total impact is quantified before the call ends** — duration, affected users or requests, and error budget consumed, while the numbers are still easy to query.

## 7. Handover to postmortem {#handover-to-postmortem}

- [ ] **A postmortem owner is named before the incident call ends** — an unassigned postmortem is a postmortem that does not get written.
- [ ] **The severity threshold that requires a postmortem is written policy** — so it is not renegotiated case by case by whoever is tired.
- [ ] **The scribe's timeline, the chat log, and relevant graphs are attached to the incident record immediately** — dashboards roll off and chat retention expires faster than postmortems get written.
- [ ] **Temporary mitigations still in place are listed explicitly for the postmortem to track to reversal.**
- [ ] **A due date for the postmortem draft is set, typically within five working days** — accuracy of recall drops sharply after the first few days.
- [ ] **Open follow-up work is ticketed now rather than deferred to the postmortem** — anything genuinely urgent should not wait for a document to be written.

## 8. Process health {#process-health}

- [ ] **The incident process is rehearsed, not only exercised in production** — run game days or tabletop exercises so that first-time commanders are not learning during a real outage.
- [ ] **New joiners shadow an incident before commanding one** — and shadowing is scheduled rather than left to chance.
- [ ] **Incident metrics are tracked over time** — time to detect, time to acknowledge, time to mitigate, and how many incidents were customer-reported rather than alert-detected.
- [ ] **Repeat incidents are counted separately** — a rising repeat rate means postmortem actions are not being completed.
- [ ] **The process documentation is short enough to be read during an incident** — a forty-page policy is not an incident process, it is an audit artefact.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Detection and declaration | | | Pass / Pass with actions / Fail |
| Triage and severity | | | Pass / Pass with actions / Fail |
| Roles | | | Pass / Pass with actions / Fail |
| Mitigation | | | Pass / Pass with actions / Fail |
| Communication cadence | | | Pass / Pass with actions / Fail |
| Resolution | | | Pass / Pass with actions / Fail |
| Handover to postmortem | | | Pass / Pass with actions / Fail |
| Process health | | | Pass / Pass with actions / Fail |

Review this table at the postmortem and raise a dated ticket with a named owner for every item that did not pass.

## Related checklists

- [Postmortem](/docs/operations/postmortem/)
- [On-Call Handover](/docs/operations/on-call-handover/)
- [Observability](/docs/operations/observability/)
- [Security Incident Response](/docs/security/incident-response/)
- [Change Management](/docs/itsm/change-management/)

## References

- [Google SRE Book — Managing Incidents](https://sre.google/sre-book/managing-incidents/)
- [Google SRE Workbook — Incident Response](https://sre.google/workbook/incident-response/)
- [PagerDuty Incident Response Documentation](https://response.pagerduty.com/)
- [Atlassian Incident Management Handbook](https://www.atlassian.com/incident-management)
- [NIST SP 800-61 Rev. 2 — Computer Security Incident Handling Guide](https://csrc.nist.gov/pubs/sp/800/61/r2/final)
