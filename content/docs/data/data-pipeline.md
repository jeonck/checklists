---
title: "Data Pipeline Review"
description: "Verify a batch or streaming pipeline is idempotent, observable, and safe to re-run before it feeds anything that matters."
icon: "alt_route"
weight: 610
toc: true
tags: ["data-engineering", "etl", "orchestration", "reliability"]
---

Most data pipelines do not fail loudly. They succeed, quietly emit the wrong numbers, and nobody notices until a finance report disagrees with itself three weeks later. This review is about making a pipeline re-runnable, correct under late and duplicated input, and observable enough that a silent failure becomes a page. Work through it before the pipeline feeds a dashboard, a model, or a downstream team.

{{< alert context="info" text="**Who runs this:** the owning data engineer plus one reviewer who consumes the output. **When:** before the pipeline is promoted to production, and again whenever its source system or partitioning scheme changes." />}}

## 1. Contracts and sources

- [ ] **Every source has a named owner and a documented delivery expectation** — what arrives, how often, and by what time, so a missing file is a broken promise rather than a surprise.
- [ ] **The read pattern from each source is agreed with its owner** — replica versus primary, snapshot versus CDC, and whether your query load is acceptable at their peak.
- [ ] **A data contract exists for each critical input** — column names, types, nullability, and semantic meaning, checked in with the pipeline rather than living in a wiki page nobody edits.
- [ ] **The pipeline pins an explicit column list rather than `SELECT *`** — a new upstream column should not silently widen your table or reorder positional consumers.
- [ ] **Source-side deletes are handled deliberately** — decide whether a row disappearing upstream means soft delete, hard delete, or ignore, because CDC and snapshot loads behave differently here.
- [ ] **Timezone and clock semantics are documented per source** — whether timestamps are UTC, local, or session-dependent, and which clock (event, ingest, or processing) each column represents.

## 2. Idempotency and reprocessing

- [ ] **Every ingestion job is idempotent on its partition key** — re-running a failed day must produce the same table state, not a second copy of the rows.
- [ ] **Writes use delete-and-insert per partition, `MERGE` on a stable key, or an atomic partition swap** — plain `INSERT` into a shared table is what turns a retry into duplicated revenue.
- [ ] **The natural key that identifies a record is defined and enforced** — a uniqueness test on that key is the cheapest duplicate detector you will ever write.
- [ ] **Re-running a task never depends on the outcome of a previous run** — no in-place counters, no reading the target table to decide what to write, no state hidden in a scratch file.
- [ ] **Output is written to a temporary location and promoted atomically** — a job that dies halfway must not leave a half-written table that a consumer can read.
- [ ] **Reprocessing a historical window is a documented, parameterised operation** — not a hand-edited copy of the DAG with the dates changed.

{{< alert context="warning" text="**Blocking:** a pipeline whose re-run duplicates rows must not go to production. Every other item here can carry a dated follow-up ticket; non-idempotent writes cannot, because the first retry silently corrupts the table." />}}

## 3. Late and out-of-order data

- [ ] **Event time and processing time are separate columns** — conflating them makes every late arrival look like it happened when you happened to read it.
- [ ] **The allowed lateness window is an explicit, documented number** — and it is derived from measured arrival distributions, not from a round number that felt safe.
- [ ] **Records arriving after the lateness window are routed somewhere visible** — a side-output table or dead-letter topic, counted and alerted on, never silently dropped.
- [ ] **Aggregates over a window are recomputed when late data lands** — or the window is explicitly declared immutable and consumers are told that late facts are lost.
- [ ] **Watermark or high-water-mark logic is tested against out-of-order input** — replay a shuffled batch in staging and assert the output matches the ordered run.
- [ ] **Restatement of already-published numbers has an agreed process** — who is told, how the correction is versioned, and whether downstream extracts are refreshed.

## 4. Schema evolution

- [ ] **Additive schema changes flow through without a manual deploy** — a new nullable upstream column should not break the load, and it should not silently vanish either.
- [ ] **Breaking changes are detected before the write, not after** — a type change or dropped column fails the run with a clear error rather than writing nulls.
- [ ] **A schema registry or equivalent enforces compatibility for streaming sources** — backward compatibility for consumers, forward compatibility for producers, chosen deliberately.
- [ ] **Column type widening is planned rather than discovered** — an integer id that becomes a string upstream will otherwise fail at the worst possible time.
- [ ] **Schema changes are versioned in the repository and reviewed like code** — including the migration for any already-written historical partitions.
- [ ] **Downstream consumers are notified before a column is renamed or removed** — with a deprecation period long enough for them to actually act.

## 5. Partitioning, storage, and layout

- [ ] **The partition column matches the dominant query filter** — partitioning by ingest date while every consumer filters on event date guarantees full scans.
- [ ] **Partition granularity is chosen against data volume** — hourly partitions on a small table produce millions of tiny files and a metadata problem worse than the scan you avoided.
- [ ] **Small-file compaction runs on a schedule for streaming or micro-batch outputs** — file count, not row count, is what eventually makes the table unqueryable.
- [ ] **File format and compression are deliberate** — columnar formats such as Parquet or ORC for analytical reads, with a codec chosen for the read/write ratio you actually have.
- [ ] **Table statistics or manifests are refreshed after each load** — a stale catalogue makes the query planner choose badly and hides newly written partitions.
- [ ] **Retention and archival are enforced by a job** — an unbounded table is a cost problem and, where personal data is involved, a compliance problem.

## 6. Orchestration, retries, and dependencies

- [ ] **Tasks declare their data dependencies rather than relying on schedule ordering** — a job that starts at 02:00 because the upstream usually finishes at 01:45 will eventually read yesterday's data.
- [ ] **Retries use bounded attempts with exponential backoff** — and retry only on transient errors, because retrying a schema violation twelve times just delays the alert.
- [ ] **A task that is already running cannot be started twice** — concurrency limits and run-key locking prevent two backfills writing the same partition simultaneously.
- [ ] **Task timeouts are set below the point at which the run blocks the next scheduled run** — a hung task with no timeout silently stops the whole schedule.
- [ ] **Sensors and polling have a deadline and a failure path** — an infinite wait for a file that will never arrive is an outage that never pages anyone.
- [ ] **The DAG can be resumed from the failed task** — a four-hour pipeline that must restart from step one after a step-nine failure will not be recoverable during an incident.
- [ ] **Credentials come from a secret manager at run time** — not from connection strings in the DAG file or variables committed to the repository.

## 7. Backfills

- [ ] **A backfill runs through the same code path as the scheduled run** — a separate backfill script drifts from production logic and produces history that does not match the present.
- [ ] **Backfills are rate-limited and resource-capped** — an unthrottled 400-day backfill will saturate the warehouse and take the daily load down with it.
- [ ] **The backfill is tested on one partition and verified before the full range is launched** — compare row counts and a few aggregates against the existing data.
- [ ] **Downstream consumers are informed before a backfill overwrites published history** — dashboards and extracts change underneath people otherwise.
- [ ] **Backfills can be paused and resumed** — and a partially completed backfill leaves the table in a consistent, identifiable state.
- [ ] **The cost of the backfill is estimated before it is run** — full-history reprocessing on a metered warehouse is one of the most common surprise invoices.

## 8. Freshness, quality gates, and monitoring

- [ ] **A freshness SLA is defined per output table** — the maximum acceptable age of the newest record, agreed with the consumers who depend on it.
- [ ] **Freshness is monitored from the table, not from the scheduler** — a green DAG run that wrote zero rows is the failure mode this catches.
- [ ] **Row-count anomaly detection is in place for each load** — an absolute floor plus a deviation band against the trailing average catches both empty and duplicated loads.
- [ ] **Quality tests run as a gate before publication, not as a report afterwards** — failing tests should stop the swap into the consumer-visible table.
- [ ] **Null rate, distinct count, and range checks are asserted on business-critical columns** — the ones that feed revenue, headcount, or a regulatory report.
- [ ] **Pipeline logs are structured and carry the run id and partition** — so an investigation starts from a query rather than from scrolling.
- [ ] **Alerts distinguish a failed run from a stale table** — they have different causes and different responders, and one can happen without the other.
- [ ] **Column-level lineage is available for critical outputs** — when a number looks wrong, tracing it to the source column should take minutes, not a day.

## 9. Cost and efficiency

- [ ] **The cost per run is measured and attributed to the pipeline** — query tags, job labels, or a dedicated warehouse make this attributable rather than a shared mystery.
- [ ] **Incremental processing is used wherever the source supports it** — full-table reloads scale linearly with history and eventually stop finishing overnight.
- [ ] **Queries prune partitions as intended** — check the query plan or bytes-scanned figure rather than assuming the filter was pushed down.
- [ ] **Compute is sized against the job rather than the largest job on the platform** — and it scales down or shuts off when idle.
- [ ] **A budget alert exists for the pipeline's warehouse or cluster** — set at a threshold that leaves time to react before month end.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Contracts and sources | | | Pass / Pass with actions / Fail |
| Idempotency and reprocessing | | | Pass / Pass with actions / Fail |
| Late and out-of-order data | | | Pass / Pass with actions / Fail |
| Schema evolution | | | Pass / Pass with actions / Fail |
| Partitioning, storage, and layout | | | Pass / Pass with actions / Fail |
| Orchestration, retries, and dependencies | | | Pass / Pass with actions / Fail |
| Backfills | | | Pass / Pass with actions / Fail |
| Freshness, quality gates, and monitoring | | | Pass / Pass with actions / Fail |
| Cost and efficiency | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with a named owner before the pipeline is scheduled in production.

## Related checklists

- [Data Quality](/docs/data/data-quality/)
- [Data Warehouse Migration](/docs/data/data-warehouse-migration/)
- [ML Model Deployment](/docs/data/ml-model-deployment/)
- [Observability](/docs/operations/observability/)
- [Database Schema Migration](/docs/development/database-schema-migration/)

## References

- [Apache Airflow documentation](https://airflow.apache.org/docs/)
- [dbt developer documentation](https://docs.getdbt.com/)
- [Apache Flink documentation](https://nightlies.apache.org/flink/flink-docs-stable/)
- [Google Cloud — Dataflow documentation](https://cloud.google.com/dataflow/docs)
- [AWS Well-Architected Framework — Data Analytics Lens](https://docs.aws.amazon.com/wellarchitected/latest/analytics-lens/analytics-lens.html)
