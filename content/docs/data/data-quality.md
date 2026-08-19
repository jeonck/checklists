---
title: "Data Quality"
description: "Verify that a dataset is accurate, complete, and trusted, with tests that block bad data rather than report it later."
icon: "rule"
weight: 620
toc: true
tags: ["data-quality", "testing", "governance", "analytics"]
---

Data quality work fails in a predictable way: a team writes a hundred tests, they all run after publication, half of them are noisy, and everyone learns to ignore the channel they post to. This checklist is about the opposite — a small set of tests that block publication, clear ownership of what "correct" means, and a way to tell whether trust in a dataset is going up or down. Use it when adopting a new critical dataset, or when a set of numbers has lost the confidence of the people who use it.

{{< alert context="info" text="**Who runs this:** the dataset owner together with the analyst or team that consumes it most. **When:** before a dataset is certified for decision-making, and quarterly for anything feeding financial or regulatory reporting." />}}

## 1. Ownership and definitions

- [ ] **Each critical dataset has a named owning team recorded in the catalogue** — an unowned table accumulates defects nobody is accountable for fixing.
- [ ] **Every business-critical metric has one written definition and one implementation** — two definitions of "active user" in two dashboards is a quality defect, not a nuance.
- [ ] **Column-level descriptions exist for critical tables and are maintained with the code** — descriptions that live only in a wiki are stale within a quarter.
- [ ] **A criticality tier is assigned per dataset** — the effort you spend on tests, SLAs, and alerting should differ between the revenue ledger and an experiment scratch table.
- [ ] **The consumers of each dataset are known** — you cannot assess the blast radius of a defect if you do not know who reads the table.
- [ ] **There is a documented path for a consumer to report a suspected defect** — and it leads to a triaged queue rather than to a chat thread that scrolls away.

## 2. Completeness

- [ ] **Row counts per load are checked against an absolute floor and a trailing deviation band** — this single test catches empty loads, partial loads, and accidental duplication.
- [ ] **Every expected partition exists for the reporting period** — a missing day is invisible in an aggregate but obvious in a partition inventory.
- [ ] **Null rates on required columns are asserted, not assumed** — including columns that are nominally nullable but should never be null in practice.
- [ ] **Reconciliation against the source system of record runs for financial and regulated data** — count and sum comparisons, on a schedule, with a tolerance agreed with finance.
- [ ] **Records rejected during load are counted, retained, and visible** — a quarantine table with zero rows is a strong signal, and a growing one is a stronger one.
- [ ] **Join fan-out is tested on critical models** — an unexpected many-to-many join inflates counts in a way that looks like growth.

## 3. Uniqueness and referential integrity

- [ ] **Each table has a declared grain and a uniqueness test on that grain** — writing down "one row per order per day" prevents most duplication bugs from surviving a deploy.
- [ ] **Foreign keys are tested for orphans against their dimension tables** — analytical warehouses usually do not enforce constraints, so the test is the constraint.
- [ ] **Deduplication logic is explicit and deterministic** — when duplicates are collapsed, the rule for which record wins must be reproducible, not dependent on file order.
- [ ] **Slowly changing dimensions have non-overlapping validity ranges** — overlapping effective dates silently double-count every fact joined through them.
- [ ] **Late-arriving dimension members are handled rather than dropped** — either a placeholder row or a documented reprocessing step, never a silent inner-join loss.

## 4. Validity, accuracy, and consistency

- [ ] **Value ranges are asserted for numeric business columns** — negative quantities, zero prices, and future-dated events are usually defects, and each one you allow should be a deliberate decision.
- [ ] **Categorical columns are tested against an accepted-values list** — a new status value appearing upstream should raise an alert, not quietly fall out of every filter.
- [ ] **Formats are validated for identifiers, currency codes, emails, and country codes** — against a standard where one exists rather than an ad-hoc regular expression.
- [ ] **Cross-field logical rules are tested** — end date after start date, total equal to the sum of its components, discount never exceeding gross value.
- [ ] **Units and currency are stored explicitly or normalised at ingest** — a mixed-currency amount column is a defect that no aggregate will ever reveal.
- [ ] **The same metric computed by two independent paths is compared periodically** — agreement between the warehouse and the operational system is the strongest accuracy signal available.

## 5. Timeliness and freshness

- [ ] **Each dataset has a published freshness SLA** — the maximum acceptable age of its newest record, agreed with consumers rather than declared by the producer.
- [ ] **Freshness is measured on the data itself, not on job success** — a successful run that wrote nothing is the case that matters.
- [ ] **Dashboards display the last-updated timestamp of their underlying data** — so a reader can see staleness without asking anybody.
- [ ] **Breaches of the freshness SLA alert the owner and are recorded** — the trend in breaches per month is a better quality metric than any single test.
- [ ] **Consumers know what to do when data is stale** — whether to wait, use the previous period, or escalate, documented alongside the dataset.

## 6. Tests, gates, and automation

- [ ] **Tests run as a gate before publication for tier-one datasets** — write to a staging location, test, then promote, so a failing test means consumers see yesterday's good data instead of today's bad data.
- [ ] **Tests are defined as code in the same repository as the transformations** — and are reviewed in the same pull request that changes the logic.
- [ ] **Each test has a severity, and only high-severity failures block or page** — a test suite where everything is critical trains people to click through.
- [ ] **Test results are stored over time, not just reported once** — you cannot tell whether quality is improving without history.
- [ ] **Anomaly-detection thresholds are reviewed after each false positive** — an alert that has cried wolf three times will be ignored on the fourth, when it is right.
- [ ] **A representative production-like sample is available for testing transformations** — anonymised or synthetic, but with the same edge cases and skew.
- [ ] **New tests are added as part of every defect fix** — a defect that recurs was a missing test, not bad luck.

{{< alert context="warning" text="**Common mistake:** running the full test suite after the table has already been published. Consumers have then already read the bad data, and the alert only tells you how many people you need to email. Gate the publication instead." />}}

## 7. Incident response for data defects

- [ ] **There is a defined severity scale for data incidents** — based on which decisions the wrong data touched, not on how many rows were affected.
- [ ] **A defect that reached consumers triggers a notification with scope and impact** — which tables, which date range, and which reports were wrong.
- [ ] **Corrected data is republished with a clear restatement note** — silently fixing a number that someone already presented externally is worse than the original defect.
- [ ] **Downstream extracts, caches, and BI tool refreshes are part of the fix** — the warehouse being correct is not the same as the dashboard being correct.
- [ ] **Data incidents get a postmortem when the impact was material** — with the missing test identified as an action item.
- [ ] **A known-issues list is visible to consumers** — trust survives a documented defect far better than a discovered one.

## 8. Governance, lineage, and access

- [ ] **Column-level lineage is available for every certified dataset** — tracing a suspect number to its source column should take minutes.
- [ ] **Personal and sensitive columns are classified and tagged in the catalogue** — quality tooling that samples data must respect those tags.
- [ ] **Access to raw sensitive columns is restricted, with masked or aggregated views for general use** — quality checks should run on the restricted side, not force wider access.
- [ ] **Certified datasets are visibly marked in the catalogue and BI tool** — otherwise consumers pick whichever table their search returned first.
- [ ] **Deprecated tables are marked, given an end date, and eventually removed** — leaving a stale duplicate table available is the most common source of two conflicting numbers.
- [ ] **Retention periods are enforced by a job and match the stated policy** — for personal data, an intention is not a control.

## 9. Measuring and improving trust

- [ ] **A small set of quality KPIs is published** — test pass rate, freshness SLA attainment, open defects, and mean time to resolution, per critical dataset.
- [ ] **Defects are counted by how they were found** — a rising share found by consumers rather than by tests means your test coverage is losing ground.
- [ ] **Coverage is tracked against the critical-column list, not the whole warehouse** — a hundred tests on unimportant tables is not coverage.
- [ ] **Quality work has an allocated share of the team's capacity** — otherwise it is always displaced by the next request.
- [ ] **The quality KPIs are reviewed with consumers, not only within the data team** — their perception of trust is the actual outcome you are managing.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Ownership and definitions | | | Pass / Pass with actions / Fail |
| Completeness | | | Pass / Pass with actions / Fail |
| Uniqueness and referential integrity | | | Pass / Pass with actions / Fail |
| Validity, accuracy, and consistency | | | Pass / Pass with actions / Fail |
| Timeliness and freshness | | | Pass / Pass with actions / Fail |
| Tests, gates, and automation | | | Pass / Pass with actions / Fail |
| Incident response for data defects | | | Pass / Pass with actions / Fail |
| Governance, lineage, and access | | | Pass / Pass with actions / Fail |
| Measuring and improving trust | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with a named owner before the dataset is certified for decision-making.

## Related checklists

- [Data Pipeline Review](/docs/data/data-pipeline/)
- [Data Warehouse Migration](/docs/data/data-warehouse-migration/)
- [ML Model Deployment](/docs/data/ml-model-deployment/)
- [Postmortem](/docs/operations/postmortem/)
- [GDPR Readiness](/docs/compliance/gdpr-readiness/)

## References

- [dbt developer documentation — tests](https://docs.getdbt.com/docs/build/data-tests)
- [Great Expectations documentation](https://docs.greatexpectations.io/)
- [DAMA-DMBOK — Data Management Body of Knowledge](https://www.dama.org/cpages/body-of-knowledge)
- [Google Cloud — Dataplex documentation](https://cloud.google.com/dataplex/docs)
- [ISO/IEC 25012 — Data quality model](https://www.iso.org/standard/35736.html)
