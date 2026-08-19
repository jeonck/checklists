---
title: "Data Warehouse Migration"
description: "Move a warehouse to a new platform without losing history, breaking reports, or paying for both systems forever."
icon: "warehouse"
weight: 640
toc: true
tags: ["data-warehouse", "migration", "reconciliation", "cutover"]
---

Warehouse migrations rarely fail on the technical load. They fail because nobody knew who used the seventeen-year-old table, because the two systems round differently and the numbers never quite matched, or because the old platform stayed switched on for three years after cutover. This checklist follows the migration in the order it actually happens — inventory, build, dual-run, reconcile, cut over, decommission — and assumes you will need to prove equivalence to people who do not trust the new system yet.

{{< alert context="info" text="**Who runs this:** the migration lead with the platform team and a representative of each major consuming group. **When:** at the start of the programme for sections 1 to 3, and then as a running gate through each wave of migrated workloads." />}}

## 1. Inventory and discovery {#inventory-and-discovery}

- [ ] **Every object in the source warehouse is inventoried with its last-accessed timestamp** — query history is the only honest source of what is really used, and it usually shows that a large share of tables are dead.
- [ ] **Consumers are identified per object from query logs, not from interviews** — the report someone forgot to mention is the one that breaks on cutover day.
- [ ] **Objects are classified into migrate, rebuild, archive, and drop** — migrating everything is how a migration becomes a three-year programme.
- [ ] **Stored procedures, scheduled jobs, and user-defined functions are catalogued** — these carry more undocumented business logic than the tables do and rarely port cleanly.
- [ ] **Every ingestion path into the warehouse is listed with its owner** — including the ad-hoc loads, the sFTP drop, and the spreadsheet someone uploads monthly.
- [ ] **Direct database connections from applications are found** — BI tools are easy to inventory, and the application with a hardcoded connection string is not.
- [ ] **Data volume and growth rate per table are measured** — they determine the transfer strategy and the target platform sizing more than any vendor calculator will.

## 2. Target design and platform decisions {#target-design-and-platform-decisions}

- [ ] **The target data model is decided deliberately rather than lifted-and-shifted** — a design built for a previous platform's distribution keys is usually wrong on the new one.
- [ ] **Type mapping is documented for every source type, especially numeric and temporal ones** — precision, scale, and timezone handling differ between platforms and quietly change results.
- [ ] **Rounding, collation, null ordering, and division semantics are tested against known cases** — these are the differences that make reconciliation fail by tiny, maddening amounts.
- [ ] **Partitioning, clustering, or distribution keys are chosen from real query patterns** — taken from the source query log rather than from assumptions.
- [ ] **Access control is designed for the target's model, not copied role by role** — a migration is the only cheap opportunity you will get to remove decades of accumulated grants.
- [ ] **Encryption, key management, and data residency requirements are confirmed for the target** — including where backups and query results are stored.
- [ ] **A cost model exists for the target with an owner and a budget alert** — consumption-based pricing punishes the query patterns that a fixed-capacity warehouse tolerated for free.

## 3. Migration approach and sequencing {#migration-approach-and-sequencing}

- [ ] **Workloads are grouped into waves by consumer, not by table** — cutting over a whole reporting domain at once is testable; cutting over half its inputs is not.
- [ ] **The first wave is a real but low-risk domain** — something with genuine consumers who will notice defects, but where being wrong for a day is survivable.
- [ ] **The historical backfill strategy is decided and sized** — full history, a bounded window, or history kept on the old platform as archive, each with a stated rationale.
- [ ] **Transfer method and network capacity have been tested with a real table** — extrapolating a terabyte-scale transfer from a small sample is how a weekend cutover becomes a fortnight.
- [ ] **Transformation logic is ported and reviewed rather than machine-translated and trusted** — automated SQL converters get syntax right and semantics subtly wrong.
- [ ] **A rollback position is defined for every wave** — the point at which you abandon the new platform for that domain, and what has to remain true for that to still be possible.
- [ ] **The freeze policy for source-side changes is agreed** — every schema change made to the old warehouse during migration must be applied to both, or dual-run will never converge.

## 4. Dual-run and dual-write {#dual-run-and-dual-write}

- [ ] **Both warehouses are loaded from the same source for the duration of the dual-run** — chaining the new warehouse off the old one hides exactly the ingestion defects you need to find.
- [ ] **The dual-run period covers at least one full business cycle** — month-end and quarter-end logic is where the undocumented adjustments live.
- [ ] **Loads into both systems use the same partition boundaries and cutoffs** — comparing a table loaded at 02:00 against one loaded at 04:00 produces differences that mean nothing.
- [ ] **Dual-write failures are handled with a documented rule** — whether a failed write to the new platform blocks the old one, and the answer is usually no, but it must be a decision.
- [ ] **The cost of running both platforms is tracked and has an agreed end date** — dual-run is a temporary expense that becomes permanent unless someone owns its termination.
- [ ] **Differences found during dual-run are logged and triaged, not fixed silently** — the pattern of differences tells you whether you have one bug or a systematic semantic mismatch.

{{< alert context="warning" text="**Blocking:** do not cut over any consumer while reconciliation differences remain unexplained. An unexplained difference is not a rounding artefact until you have proven it is one, and every migration that skipped this step spent the following year defending its numbers." />}}

## 5. Reconciliation and validation {#reconciliation-and-validation}

- [ ] **Row counts match per table and per partition** — a whole-table count can match while individual days are shifted by a timezone bug.
- [ ] **Checksums or aggregate hashes are compared per partition** — sum, min, max, and count of distinct values on key columns catch what a row count cannot.
- [ ] **Numeric columns are compared with an explicitly agreed tolerance** — and any tolerance above zero is signed off by the business owner of the number, not by the engineer.
- [ ] **Full row-level comparison is run on the highest-value tables** — a set difference in both directions, since extra rows and missing rows have different causes.
- [ ] **Critical reports are run on both platforms and compared output to output** — the report is the contract with the consumer, and matching tables do not guarantee matching reports.
- [ ] **Null handling and empty-string differences are tested specifically** — several platforms disagree on whether an empty string is null, which silently changes joins and counts.
- [ ] **Reconciliation is automated and runs on every dual-run cycle** — a one-off manual comparison is stale the day after it is produced.
- [ ] **Query performance on the target is compared against the source for representative workloads** — a correct warehouse that is slower than the one it replaced will not be accepted.

## 6. Consumer cutover {#consumer-cutover}

- [ ] **Each consumer's cutover is a scheduled event with a named owner on their side** — not an announcement that the new connection details are available.
- [ ] **BI tool connections, extracts, and caches are repointed and refreshed as part of the cutover** — a dashboard reading a stale extract will show old numbers long after the warehouse moved.
- [ ] **A read-only alias or view layer insulates consumers from physical table names** — so future changes do not require another round of consumer coordination.
- [ ] **Service accounts and application connection strings are migrated through configuration, not code edits** — and the credentials come from a secret manager.
- [ ] **Consumers are given a documented verification step and a window to confirm** — a short checklist of numbers they can check themselves is worth more than an all-clear email.
- [ ] **Access rights on the target are verified per consumer before cutover** — including row-level and column-level restrictions that existed on the source.
- [ ] **Training and updated documentation are delivered before the cutover, not after** — dialect differences will otherwise generate a wave of support requests.
- [ ] **The old platform is switched to read-only for migrated domains immediately after cutover** — leaving both writable is how the two systems start diverging permanently.

## 7. Operational readiness of the target {#operational-readiness-of-the-target}

- [ ] **Backup, restore, and time-travel settings are configured and a restore has been performed** — an untested backup on a new platform is an untested backup.
- [ ] **Monitoring covers freshness, load failures, query errors, and cost** — with the same alert routes as the rest of the estate rather than a new tool nobody watches.
- [ ] **Disaster recovery objectives for the target are documented and achievable** — measured on the new platform, since the old figures do not transfer.
- [ ] **Runbooks are updated for the new platform** — how to rerun a load, how to grant access, how to investigate a slow query.
- [ ] **On-call responsibility for the target warehouse is assigned and the rotation knows it** — a platform in production without an owner defaults to the migration team indefinitely.
- [ ] **Audit logging of access to sensitive data is enabled and its retention meets policy** — verify it before cutover, since retrofitting an audit trail for a past period is impossible.
- [ ] **Workload isolation prevents an ad-hoc query from starving the scheduled loads** — separate warehouses, queues, or resource groups, with limits set.

## 8. Decommissioning the source {#decommissioning-the-source}

- [ ] **A decommission date is agreed and communicated before cutover, not after** — without a date, the old warehouse survives on the strength of unspecified fear.
- [ ] **Query activity on the source is monitored to zero before shutdown** — the log tells you whether anyone is still connecting, and someone always is.
- [ ] **A final archive of the source is taken in an open, readable format** — with a documented and tested restore path, because a proprietary backup nobody can open is not an archive.
- [ ] **Legal and regulatory retention obligations for the archived data are confirmed** — and the archive's retention is enforced by a policy rather than by neglect.
- [ ] **The source is made read-only for a defined quiet period before deletion** — a reversible step between cutover and permanent loss.
- [ ] **Licences, reserved capacity, and support contracts are cancelled** — the savings that justified the migration are only realised at this step.
- [ ] **Credentials, service accounts, network rules, and integrations pointing to the source are removed** — a decommissioned platform left reachable is an unpatched, unmonitored attack surface.
- [ ] **The migration is closed with a written record of what moved, what was dropped, and where the archive lives** — the question will be asked years later by someone who was not there.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Inventory and discovery | | | Pass / Pass with actions / Fail |
| Target design and platform decisions | | | Pass / Pass with actions / Fail |
| Migration approach and sequencing | | | Pass / Pass with actions / Fail |
| Dual-run and dual-write | | | Pass / Pass with actions / Fail |
| Reconciliation and validation | | | Pass / Pass with actions / Fail |
| Consumer cutover | | | Pass / Pass with actions / Fail |
| Operational readiness of the target | | | Pass / Pass with actions / Fail |
| Decommissioning the source | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with a named owner before the wave is declared complete.

## Related checklists

- [Data Pipeline Review](/docs/data/data-pipeline/)
- [Data Quality](/docs/data/data-quality/)
- [Cloud Migration](/docs/cloud/cloud-migration/)
- [Database Schema Migration](/docs/development/database-schema-migration/)
- [Backup and Recovery](/docs/operations/backup-and-recovery/)

## References

- [Google Cloud — BigQuery documentation](https://cloud.google.com/bigquery/docs)
- [AWS Database Migration Service User Guide](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html)
- [Amazon Redshift Management Guide](https://docs.aws.amazon.com/redshift/latest/mgmt/welcome.html)
- [Snowflake Documentation](https://docs.snowflake.com/)
- [Microsoft Cloud Adoption Framework for Azure](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/)
