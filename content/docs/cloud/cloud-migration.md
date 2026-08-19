---
title: "Cloud Migration Cutover"
description: "Verify a workload can be moved to the cloud and switched over safely, with a rehearsed rollback."
icon: "swap_horiz"
weight: 330
toc: true
tags: ["migration", "cutover", "cloud", "runbook"]
---

Migration projects rarely fail at the technology; they fail in the hour when traffic actually moves. This checklist is ordered as a timeline rather than by topic — assessment, dependency mapping, target build, rehearsal, freeze, the cutover window itself, validation, and rollback — so it can be worked through as the plan takes shape and then read top to bottom on the day. Adapt the timings to your change window, but do not reorder the phases.

{{< alert context="info" text="**Who runs this:** the migration lead, with the workload owner, a database engineer, and a network engineer. **When:** start at least four weeks before the cutover date; the rehearsal section must complete at least one week before." />}}

## 1. Assessment and scope {#assessment-and-scope}

- [ ] **The migration strategy is chosen and written down per component** — rehost, replatform, refactor, repurchase, retire, or retain; a project that has not decided will drift into a partial refactor under time pressure.
- [ ] **The scope boundary lists what is explicitly not moving in this window** — the ambiguity between "later" and "never" is where orphaned dependencies live.
- [ ] **A current-state performance baseline is captured** — request rate, latency percentiles, database query profile, batch job durations, and peak-day figures, because "it feels slower" is unarguable without numbers.
- [ ] **Data volumes and growth rate are measured, not estimated** — total size, row counts of the largest tables, and daily change rate, since the change rate determines whether an offline copy can ever catch up.
- [ ] **Data residency, sovereignty, and regulatory constraints on the target region are confirmed in writing** — moving personal data across a jurisdiction boundary is a legal decision, not an architectural one.
- [ ] **Success criteria and the acceptance owner are agreed before work starts** — including the latency and error-rate thresholds that define a successful cutover.
- [ ] **The rollback deadline within the window is fixed and agreed** — the clock time after which you stop trying to fix forward and revert, decided when everyone is calm.

## 2. Dependency and integration mapping {#dependency-and-integration-mapping}

- [ ] **Every inbound caller of the workload is identified, including batch and internal tooling** — a report generator that runs monthly is the classic caller nobody remembers until it fails.
- [ ] **Every outbound dependency is identified with its protocol, endpoint, and authentication method.**
- [ ] **Network traffic has been observed to confirm the diagram** — flow logs, connection tables, or a packet capture over a full business cycle, because the architecture diagram is a description of intent.
- [ ] **Hard-coded IP addresses and hostnames in configuration, scripts, and firewall rules are catalogued** — these are the single most common cause of a partially working cutover.
- [ ] **Third-party allow-lists that reference your current egress IPs are identified and updated ahead of time** — partner firewall changes have their own lead time, often weeks.
- [ ] **Shared infrastructure dependencies are flagged** — an on-premises directory service, message broker, or file share that stays behind creates a latency and availability coupling that must be designed for explicitly.
- [ ] **Scheduled jobs, cron entries, and their timezone assumptions are inventoried** — jobs that double-run during cutover, or silently stop, are a frequent post-migration data problem.

## 3. Target environment build and data replication {#target-environment-build-and-data-replication}

- [ ] **The target environment is built from code, not by hand** — you will need to build it at least twice, once for rehearsal and once for real.
- [ ] **Target capacity is sized from the baseline in section 1, with headroom for the cutover spike** — cold caches and reconnecting clients make the first hour the busiest.
- [ ] **A load test on the target has reached the baseline peak plus a margin** — and the breaking point is known.
- [ ] **Continuous replication is running and lag is monitored and graphed** — a replication tool that reports healthy while lag grows linearly is discovered too late otherwise.
- [ ] **Schema, collation, character set, and timezone settings match or the differences are documented** — a collation mismatch changes sort order and unique constraint behaviour without any error.
- [ ] **Objects that replication tools do not carry have been handled explicitly** — sequences, triggers, stored procedures, grants, scheduled events, and foreign key validation state.
- [ ] **Large object and file storage is pre-seeded with a delta sync planned for the window** — copying terabytes during the outage is how a two-hour window becomes an eight-hour one.
- [ ] **Secrets, connection strings, and configuration exist in the target and have been used successfully at least once** — an untested secret is an outage waiting for the worst moment.
- [ ] **Observability is live on the target before cutover, with the same dashboards as the source** — you must be able to compare the two systems side by side during validation.

## 4. Rehearsal and dry run {#rehearsal-and-dry-run}

- [ ] **A full dry run of the cutover runbook has been executed end to end** — with a copy of production data, and against the real target environment.
- [ ] **Every step in the runbook has an owner, an expected duration, and an explicit verification** — a step that says only "switch DNS" will be performed differently by each person who reads it.
- [ ] **The dry run was timed and the total fits inside the change window with at least 50% margin** — real cutovers are slower than rehearsals, and the margin is what pays for the surprise.
- [ ] **Rollback was rehearsed, not just documented** — reverting is the path you will take under maximum stress, so it is the path that most needs practice.
- [ ] **Data validation queries were run and their expected outputs recorded** — row counts, checksums on key tables, and business-level aggregates such as yesterday's order total.
- [ ] **The dry run surfaced defects and each has a fix or an accepted workaround** — a dry run that found nothing was not a real dry run.
- [ ] **The communication plan was rehearsed** — who posts status, where, and at what interval, including the message for customers.

{{< alert context="danger" text="**Blocking:** do not proceed to a production cutover without at least one complete, timed dry run including rollback. If the schedule does not allow a rehearsal, the schedule is wrong, not the checklist." />}}

## 5. Freeze and go/no-go {#freeze-and-go-no-go}

- [ ] **A change freeze is in effect on both source and target for an agreed period before the window** — including schema changes, configuration edits, and third-party integrations.
- [ ] **The freeze is enforced technically where possible** — pipeline gate or branch protection, because a freeze that relies on everyone remembering does not hold.
- [ ] **DNS time-to-live has been lowered well in advance and the reduction has propagated** — lowering TTL from 3600 to 60 an hour before cutover does nothing, since resolvers still hold the old record for the old TTL.
- [ ] **A go/no-go meeting has occurred with named decision-makers and a documented decision** — replication lag, open defects, staffing, and dependency readiness are the standing agenda.
- [ ] **On-call staffing is confirmed for the window and for the following business day** — including database, network, and application specialists, and a decision-maker with authority to call the rollback.
- [ ] **Vendor and cloud provider support tickets are pre-opened for the window where the platform offers it** — you do not want to be explaining your architecture to a first-line agent at 02:00.
- [ ] **Automated alerting that will fire spuriously during the window is muted deliberately, with an unmute time set** — and the mute must not cover the alerts you need to see.

## 6. The cutover window {#the-cutover-window}

- [ ] **A single bridge call or channel is open and all participants are on it before step one.**
- [ ] **The runbook is being followed in a shared document with steps ticked and timestamped live** — this is your incident timeline if something goes wrong, and your evidence for the retrospective.
- [ ] **Source traffic is stopped or drained in the agreed way** — read-only mode, maintenance page, or queue pause; and it is confirmed stopped by observation, not assumption.
- [ ] **Final data delta is applied and replication lag is confirmed at zero before the source is closed** — cutting over with residual lag silently loses the transactions in flight.
- [ ] **The source is made non-writable before the target is made writable** — a window where both accept writes produces a split-brain that is expensive and sometimes impossible to reconcile.
- [ ] **Traffic is switched at the agreed layer** — DNS, load balancer target group, or connection string; and the mechanism was the one rehearsed.
- [ ] **Each step's verification is performed before moving to the next** — the temptation to run ahead during a tight window is exactly how a failure is discovered three steps too late.

## 7. Post-cutover validation {#post-cutover-validation}

- [ ] **Data validation queries from the rehearsal are re-run and match expected outputs** — row counts, checksums, and business aggregates, before any user traffic is admitted.
- [ ] **Smoke tests cover the critical user journeys end to end, including a real write** — a read-only health check will pass against a database that cannot accept writes.
- [ ] **Authentication and authorisation are verified for each user class** — including federated sign-in, service accounts, and any API client with an IP allow-list.
- [ ] **Integrations are confirmed by observing real traffic in both directions** — payment provider, email delivery, webhooks, and file transfers; a webhook endpoint that changed hostname will fail quietly at the sender.
- [ ] **Scheduled and batch jobs are confirmed to have run once successfully, or confirmed disabled deliberately** — the first overnight batch is the most common post-migration incident.
- [ ] **Latency and error rate are compared against the section 1 baseline, not against zero** — and any regression beyond the agreed threshold triggers the acceptance decision, not a shrug.
- [ ] **A defined soak period is observed before the window is declared closed** — long enough to include one peak traffic period.

## 8. Rollback {#rollback}

- [ ] **The rollback trigger conditions are written down and objective** — specific error-rate, data-integrity, or elapsed-time thresholds, so the decision is not a debate at 04:00.
- [ ] **The rollback procedure is a step-by-step runbook, not a paragraph, and was executed in the dry run.**
- [ ] **Reverse replication or a documented reconciliation path exists for data written to the target** — without it, rollback means losing every transaction since cutover, which is often unacceptable and turns rollback into a fiction.
- [ ] **The source environment is left intact and running until the retention decision point** — deleting or repurposing source infrastructure on cutover day removes your only real safety net.
- [ ] **DNS and load balancer changes are reversible within the lowered TTL** — and the reverse change has been prepared in advance rather than typed under pressure.
- [ ] **Stakeholder communication for a rollback is pre-drafted** — writing it during the rollback delays the rollback.

## 9. Stabilisation and decommissioning {#stabilisation-and-decommissioning}

- [ ] **Hypercare support is staffed for an agreed period after cutover** — typically one full business cycle, including the first month-end if any process depends on it.
- [ ] **DNS TTL is returned to its normal value once the migration is confirmed stable.**
- [ ] **Monitoring thresholds and autoscaling parameters are retuned to the target's actual behaviour** — thresholds carried across from the old platform will be wrong in both directions.
- [ ] **Cost on the target is reviewed against the estimate within the first full billing period** — migrations very commonly land 30% over forecast, and the causes are fixable if caught early.
- [ ] **Source infrastructure decommissioning is scheduled with a data retention decision and an owner** — including final backups, licence returns, and contract termination notice periods.
- [ ] **Documentation, runbooks, and architecture diagrams are updated to describe the target** — the pre-migration runbook is now actively misleading.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Assessment and scope | | | Pass / Pass with actions / Fail |
| Dependency and integration mapping | | | Pass / Pass with actions / Fail |
| Target environment build and data replication | | | Pass / Pass with actions / Fail |
| Rehearsal and dry run | | | Pass / Pass with actions / Fail |
| Freeze and go/no-go | | | Pass / Pass with actions / Fail |
| The cutover window | | | Pass / Pass with actions / Fail |
| Post-cutover validation | | | Pass / Pass with actions / Fail |
| Rollback | | | Pass / Pass with actions / Fail |
| Stabilisation and decommissioning | | | Pass / Pass with actions / Fail |

Any "Fail" in sections 1 to 5 is a no-go for the cutover date; record every "Pass with actions" as a dated ticket with an owner.

## Related checklists

- [Change Management](/docs/itsm/change-management/)
- [Database Schema Migration](/docs/development/database-schema-migration/)
- [DNS Migration](/docs/networking/dns-migration/)
- [Release Day](/docs/devops/release-day/)
- [Backup and Recovery](/docs/operations/backup-and-recovery/)

## References

- [AWS Prescriptive Guidance — Migration](https://docs.aws.amazon.com/prescriptive-guidance/latest/strategy-migration/welcome.html)
- [Azure Cloud Adoption Framework — Migrate](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/migrate/)
- [Google Cloud — Migration to Google Cloud](https://cloud.google.com/architecture/migration-to-gcp-getting-started)
- [AWS Well-Architected Framework — Operational Excellence Pillar](https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/welcome.html)
