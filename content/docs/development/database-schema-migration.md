---
title: "Database Schema Migration"
description: "Verify a schema change deploys, rolls back, and survives a rolling release without locking or losing data."
icon: "schema"
weight: 430
toc: true
tags: ["database", "migration", "deployment", "backend"]
---

A schema migration is the one deployment step that is not trivially reversible. Code can be rolled back in seconds; a dropped column cannot. The two failure modes that cause almost every migration incident are the same every time: a lock held long enough to stall the application, and a schema change deployed in the wrong order relative to the code that reads it. This checklist is built around avoiding both.

{{< alert context="info" text="**Who runs this:** the engineer writing the migration, reviewed by someone who has operated this database in production. **When:** before the migration is merged, and again during the release window planning." />}}

## 1. Expand and contract planning

- [ ] **The change is decomposed into expand, migrate, and contract phases** — expand adds the new structure without touching the old, the application is moved over, and only then does contract remove the old structure. Every phase ships and is verified separately.
- [ ] **Expand-phase changes are purely additive** — new nullable columns, new tables, new indexes. Nothing existing is renamed, retyped, dropped, or given a constraint that current writes could violate.
- [ ] **A rename is never done in place** — add the new column, dual-write to both, backfill, switch reads, stop writing the old, then drop it. An in-place rename breaks every running instance of the old code the instant it commits.
- [ ] **Dual-write logic is behind a flag with a defined removal date** — it is temporary scaffolding, and a permanent dual-write is a permanent source of divergence between two columns.
- [ ] **The contract phase is a separate deployment, after the previous release is confirmed irreversible** — dropping a column while a rollback to the previous version is still plausible converts one incident into two.
- [ ] **Each phase has been checked for whether it can be safely paused indefinitely** — real releases stall, and every intermediate state must be a state you can live in for a week.
- [ ] **The full sequence is written down in order, with the deploy that accompanies each step** — this is the artefact the release runbook needs, not the migration file itself.

## 2. Backward compatibility during rolling deploys

- [ ] **Old code runs correctly against the new schema** — during a rolling deploy, and for the entire duration of any canary, both versions are live at once and both are serving writes.
- [ ] **New code runs correctly against the old schema** — the migration and the deploy are not atomic, and either can land first depending on your pipeline.
- [ ] **New columns are nullable or have a database default** — adding a `NOT NULL` column with no default fails immediately for the old code, which does not know to supply it.
- [ ] **`SELECT *` is not used on tables being changed** — column addition or reordering silently changes the shape of the result set for ORM code that positionally maps rows.
- [ ] **New constraints are added as not-valid first, then validated separately** — on PostgreSQL, `ADD CONSTRAINT ... NOT VALID` followed by `VALIDATE CONSTRAINT` avoids a full-table scan under an exclusive lock.
- [ ] **Enum and lookup values are added before any code emits them, and removed only after no code emits them** — this is the same expand/contract discipline applied to data.
- [ ] **Application-level compatibility is actually tested, not reasoned about** — run the previous release's test suite against the migrated schema in CI.

{{< alert context="danger" text="**Blocking:** a migration that renames or drops a column in the same release as the code change cannot be rolled back. The old code will fail on every request the moment the migration commits, and re-adding the column does not restore the data. Split it into expand and contract." />}}

## 3. Locking and large tables

- [ ] **The lock each statement takes has been looked up for your specific engine and version** — the answer differs between PostgreSQL and MySQL, and between versions of each; assume nothing from experience on the other one.
- [ ] **The migration acquires no lock that blocks writes for more than a second or two on production-sized data** — a table rewrite on a large table stalls every connection queued behind it, exhausts the pool, and takes the application down even though the database is technically healthy.
- [ ] **Indexes on large tables are built concurrently** — `CREATE INDEX CONCURRENTLY` on PostgreSQL, or the engine's online DDL equivalent, and note that concurrent index builds cannot run inside a transaction block.
- [ ] **A `lock_timeout` (or equivalent) is set so the migration fails fast rather than queueing** — an ungranted `ACCESS EXCLUSIVE` lock request blocks every subsequent query on that table, including reads, turning a slow migration into a total outage.
- [ ] **Long-running transactions and idle-in-transaction sessions are checked for before starting** — a single open transaction from an analytics query will block the lock acquisition indefinitely.
- [ ] **The migration has been timed against a production-sized copy** — the row count in staging is not evidence about anything.
- [ ] **An online schema change tool is used where the engine cannot do it natively** — and its trigger or replication overhead has been accounted for in the maintenance window.
- [ ] **Replication lag impact is estimated** — a migration that replays serially on replicas can push read replicas minutes behind and break read-after-write assumptions.

## 4. Backfills

- [ ] **Backfill is separated from the DDL migration** — mixing a multi-million-row `UPDATE` into a schema migration holds locks and blocks the deployment pipeline.
- [ ] **The backfill runs in bounded batches with a pause between them** — commit each batch, keep transactions short, and leave headroom for production traffic.
- [ ] **The backfill is resumable and idempotent** — it will be interrupted, and restarting from zero on a large table is often not affordable.
- [ ] **Backfill progress is observable** — rows remaining, current rate, and estimated completion, so someone can decide whether to let it run or stop it.
- [ ] **The backfill has a kill switch and its effect on database load is monitored while it runs** — watch replication lag, lock waits, and IO saturation, not just CPU.
- [ ] **New writes populate the new column while the backfill is in progress** — otherwise the backfill can never converge.
- [ ] **Completion is verified by a count of remaining unpopulated rows, not by the job exiting zero.**

## 5. Rollback safety

- [ ] **The rollback path is written and tested, not assumed** — for each migration, either a tested down-migration exists, or the migration is documented as forward-only with the reason.
- [ ] **Every step is classified as reversible or irreversible before the release** — dropping a column, dropping a table, and narrowing a column type are irreversible regardless of what the down-migration file claims.
- [ ] **A down-migration that recreates a dropped column is marked as data-lossy** — restoring the structure without the data usually leaves the application worse off than the failure it was meant to fix.
- [ ] **A backup or snapshot taken immediately before the migration is verified to exist** — and the restore time for the current data volume is known and acceptable.
- [ ] **The rollback plan states what happens to data written by the new version** — if the new code wrote rows the old code cannot interpret, rolling the code back does not roll the data back.
- [ ] **Rollback has been rehearsed at least once on a non-production environment with production-like data.**
- [ ] **The point of no return is identified explicitly in the runbook** — the step after which rolling forward is the only option, so nobody discovers it during an incident.

## 6. Correctness of the change itself

- [ ] **Column types match the data's real domain** — check maximum length, precision for decimals, and whether a 32-bit integer key will exhaust; an identifier column approaching its maximum is a well-known outage cause.
- [ ] **Timestamps use a timezone-aware type and store UTC** — retrofitting timezone handling onto a populated column is a migration nobody enjoys.
- [ ] **Foreign keys and their cascade behaviour are deliberate** — `ON DELETE CASCADE` on a large table can delete far more than intended and hold locks while doing it.
- [ ] **Every new index is justified by a query, and the query plan has been checked with `EXPLAIN` on realistic data** — redundant indexes slow every write and consume storage forever.
- [ ] **Uniqueness that the application relies on is enforced by a database constraint** — application-level checks lose to concurrency.
- [ ] **Default values are considered for their effect on existing rows** — on older engines, adding a column with a default rewrites the whole table.
- [ ] **Character set and collation match the rest of the schema** — a mismatched collation on a join column silently disables index usage.

## 7. Testing and review

- [ ] **The migration has been run against a restored copy of production data** — anonymised if necessary, but with real row counts and real data distribution.
- [ ] **The migration runs cleanly from an empty database as well** — the whole migration history must still build a working schema for new environments and for CI.
- [ ] **Running the migration twice is safe or is prevented** — a partially applied migration will be retried, and the retry must not fail on the half that succeeded.
- [ ] **The migration is reviewed by someone who has operated this database** — schema review is a different skill from code review, and reviewers who only read the diff miss lock behaviour entirely.
- [ ] **Automated linting for unsafe migration patterns runs in CI** — several tools catch non-concurrent index creation, in-place renames, and `NOT NULL` additions without defaults.
- [ ] **The generated DDL has been read, not just the ORM migration source** — framework migration DSLs regularly emit something other than what the author intended.

```sql
-- Expand: additive, no rewrite, no blocking lock.
ALTER TABLE orders ADD COLUMN customer_uuid uuid;
CREATE INDEX CONCURRENTLY idx_orders_customer_uuid ON orders (customer_uuid);

-- Later release, after the backfill has converged and reads have switched:
ALTER TABLE orders
  ADD CONSTRAINT orders_customer_uuid_not_null
  CHECK (customer_uuid IS NOT NULL) NOT VALID;
ALTER TABLE orders VALIDATE CONSTRAINT orders_customer_uuid_not_null;
```

## 8. Execution and monitoring

- [ ] **The execution window avoids peak traffic and any scheduled batch job** — including the nightly export nobody remembers owning.
- [ ] **Someone with production database access is present while the migration runs** — not merely reachable.
- [ ] **Lock waits, active query count, replication lag, and error rate are watched live during execution** — with an agreed threshold at which the migration is aborted.
- [ ] **The abort procedure is known before starting** — how to cancel the statement, and how to terminate the backend if cancelling does not work.
- [ ] **Migrations run as a distinct database role from the application** — the application should not hold DDL privileges at runtime.
- [ ] **The migration is applied by the pipeline, not by hand from a laptop** — with the exception path documented for emergencies.
- [ ] **Post-migration verification is defined in advance** — the specific queries, counts, and dashboards that confirm success, checked before the change is declared done.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Expand and contract planning | | | Pass / Pass with actions / Fail |
| Backward compatibility during rolling deploys | | | Pass / Pass with actions / Fail |
| Locking and large tables | | | Pass / Pass with actions / Fail |
| Backfills | | | Pass / Pass with actions / Fail |
| Rollback safety | | | Pass / Pass with actions / Fail |
| Correctness of the change itself | | | Pass / Pass with actions / Fail |
| Testing and review | | | Pass / Pass with actions / Fail |
| Execution and monitoring | | | Pass / Pass with actions / Fail |

Anything in the rollback safety or locking sections that is not a clear pass should block the release rather than becoming a follow-up ticket.

## Related checklists

- [Release Day](/docs/devops/release-day/)
- [Backup and Recovery](/docs/operations/backup-and-recovery/)
- [Production Readiness Review](/docs/devops/production-readiness/)
- [Code Review](/docs/development/code-review/)
- [Change Management](/docs/itsm/change-management/)

## References

- [PostgreSQL — ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html)
- [PostgreSQL — Explicit Locking](https://www.postgresql.org/docs/current/explicit-locking.html)
- [MySQL — Online DDL Operations](https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-operations.html)
- [Martin Fowler — Parallel Change (expand/contract)](https://martinfowler.com/bliki/ParallelChange.html)
- [Martin Fowler — Evolutionary Database Design](https://martinfowler.com/articles/evodb.html)
