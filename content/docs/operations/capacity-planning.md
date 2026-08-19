---
title: "Capacity Planning"
description: "Verify a service has measured headroom, credible demand forecasts, and limits that fail safely under load."
icon: "trending_up"
weight: 560
toc: true
tags: ["capacity", "performance", "scalability", "sre"]
---

Capacity planning is the discipline of knowing, before your users find out, where the system breaks and how long you have until you get there. It sits between load testing and cost management: measure what you have, forecast what you will need, and make sure the difference is bought or engineered with time to spare. This checklist follows that order.

{{< alert context="info" text="**Who runs this:** the service owner with input from finance and whoever owns the infrastructure budget. **When:** quarterly, before any expected demand event, and after any architectural change that alters the constraining resource." />}}

## 1. Demand model

- [ ] **The unit of demand is defined in business terms** — orders per minute, active sessions, or documents indexed, so that a product forecast translates directly into infrastructure.
- [ ] **The peak-to-average ratio is measured, not assumed** — provisioning to the daily average guarantees failure at the daily peak, which is the only hour that matters.
- [ ] **Seasonal and weekly patterns are documented** — month-end batch runs, payday spikes, and campaign traffic each have a shape that averages hide entirely.
- [ ] **Known future events are on a calendar with expected multipliers** — marketing launches, partner integrations, migrations, and regulatory deadlines all have owners who can be asked.
- [ ] **Growth is forecast from at least twelve months of history where it exists** — a trend fitted to one quarter will mislead in either direction.
- [ ] **Internal demand is modelled alongside customer demand** — analytics jobs, backfills, and your own retrying clients frequently dominate the peak.
- [ ] **The forecast records its assumptions explicitly** — so that when reality diverges you can tell which assumption was wrong rather than rebuilding it from scratch.

## 2. Current utilisation and the constraining resource

- [ ] **The constraining resource is identified per service** — CPU, memory, disk IOPS, network, database connections, or a third-party rate limit; everything else is secondary.
- [ ] **Utilisation is measured at peak, not averaged over the day** — an instance at 40% daily average can be saturated for two hours every evening.
- [ ] **Utilisation is tracked per instance as well as in aggregate** — an unbalanced shard or a hot partition saturates while the fleet average looks comfortable.
- [ ] **Resources that are consumed monotonically have their own tracking** — disk usage, table sizes, and index growth only ever go one way and produce entirely predictable outages.
- [ ] **Connection pools and thread pools are counted as capacity** — database connection exhaustion is a far more common ceiling than raw CPU.
- [ ] **Provider quotas and service limits are inventoried** — API rate limits, instances per region, IP addresses, and load balancer rules all become the binding constraint eventually.
- [ ] **Efficiency is tracked as demand per unit of resource** — this is what tells you whether growth in cost is growth in traffic or growth in waste.

## 3. Headroom and safety margins

- [ ] **A target headroom is defined and justified** — enough to absorb the loss of an availability zone plus the largest plausible traffic spike, not a number copied from another team.
- [ ] **Headroom accounts for the failure of the largest single unit** — losing one of three zones means the surviving two must carry the full peak.
- [ ] **The time required to add capacity is measured** — if provisioning takes two hours, headroom must cover two hours of growth plus the detection delay.
- [ ] **Alerts fire on projected exhaustion, not only on current utilisation** — an alert saying disk will be full in six days is actionable; one saying disk is 95% full is an incident.
- [ ] **Headroom is validated after each scaling change** — reducing instance count for cost reasons quietly consumes the margin that was there for zone failure.
- [ ] **Stateful components have separate, larger margins** — adding a database replica or resharding takes hours or days, unlike adding a stateless instance.

## 4. Load testing and the breaking point

- [ ] **Load tests run against production-like infrastructure** — same instance classes, same database tier, same network topology, or the results describe a system you do not operate.
- [ ] **The test uses a realistic traffic mix** — replayed or modelled from production, since a single-endpoint benchmark tells you nothing about contention between endpoints.
- [ ] **The breaking point is known as a number** — the request rate at which latency crosses the SLO, and which component fails first when you get there.
- [ ] **Behaviour beyond the breaking point is characterised** — the system should degrade and shed load rather than collapse into a retry storm it cannot recover from.
- [ ] **Recovery from overload is tested, not just the overload itself** — many systems survive the spike and then fall over on the queued backlog when traffic returns to normal.
- [ ] **Soak tests run long enough to reveal leaks** — memory growth, file descriptor exhaustion, and connection leaks appear over hours, not over a ten-minute test.
- [ ] **Load test results are dated and re-run after significant changes** — a benchmark from two releases ago describes a system that no longer exists.

{{< alert context="warning" text="**Common mistake:** load testing one service in isolation. The interesting failures happen at shared dependencies — the database, the cache, the identity provider — under the combined load of every caller, which single-service tests never produce." />}}

## 5. Scaling mechanisms

- [ ] **Autoscaling triggers on the constraining resource** — scaling on CPU when the bottleneck is database connections adds load to the thing that is already failing.
- [ ] **Scale-up is fast and scale-down is slow** — aggressive scale-down causes thrashing and leaves you short at the start of the next spike.
- [ ] **Maximum scaling limits are set and are affordable** — check the ceiling against the monthly budget, not only against the technical limit.
- [ ] **The system has been observed actually scaling under load** — autoscaling configuration is frequently wrong in ways only a real scale event reveals.
- [ ] **Cold start and warm-up time are accounted for** — an instance that takes three minutes to become useful cannot respond to a sixty-second spike.
- [ ] **Scaling one tier does not overwhelm another** — more application instances mean more database connections, and the database rarely scales on the same timescale.
- [ ] **Manual scaling procedures exist for when autoscaling fails or is too slow** — documented, and tested by someone who is not the person who wrote them.

## 6. Protection under overload

- [ ] **Rate limiting protects the service from any single client** — including your own internal callers and your own retrying clients, which are the usual culprits.
- [ ] **Load shedding drops the least important work first** — a documented priority order lets you keep checkout working while analytics queries are rejected.
- [ ] **Backpressure propagates rather than being absorbed silently** — an unbounded internal queue converts an overload into an unbounded latency increase and eventually a memory failure.
- [ ] **Retries have a budget and use exponential backoff with jitter** — synchronised naive retries turn a brief slowdown into a self-inflicted denial of service.
- [ ] **Circuit breakers protect against a slow dependency** — timeouts alone still consume every thread while waiting.
- [ ] **Rejection under load is fast and cheap** — a request that is going to fail should fail immediately rather than consuming a connection for thirty seconds first.

## 7. Cost and procurement

- [ ] **Capacity cost per unit of demand is calculated** — cost per thousand requests or per active user is the number that makes a forecast financially meaningful.
- [ ] **The forecast is translated into a budget and shared with finance before the spend arrives** — capacity surprises are usually organisational failures rather than technical ones.
- [ ] **Committed-use discounts and reservations are sized against the forecast floor, not the peak** — over-committing is as costly as not committing at all.
- [ ] **Lead times for anything not instantly available are known** — GPU capacity, reserved instances, licences, and physical hardware all have procurement delays measured in weeks.
- [ ] **Budget alerts are configured at thresholds that leave time to react** — an alert at 100% of budget is a report, not a warning.
- [ ] **Idle and over-provisioned resources are reviewed regularly** — waste consumes the budget that would otherwise fund genuine headroom.

## 8. Review cadence and ownership

- [ ] **Capacity is reviewed on a fixed schedule with a named owner** — quarterly for most services, monthly for anything growing quickly.
- [ ] **Forecast accuracy is measured against actuals each cycle** — a forecast that is never scored never improves.
- [ ] **A dashboard shows current utilisation against forecast demand and the breaking point** — so headroom is visible without anyone having to assemble it.
- [ ] **Product roadmap changes trigger a capacity review** — a new feature that multiplies read volume needs to be known before it ships, not after.
- [ ] **Capacity findings feed the risk register with owners and dates** — an exhaustion date more than one review cycle away is a plan; one inside the cycle is an incident waiting.
- [ ] **On-call engineers know the current headroom and where the limits are** — this is the context that turns an ambiguous latency page into a fast diagnosis.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Demand model | | | Pass / Pass with actions / Fail |
| Current utilisation and the constraining resource | | | Pass / Pass with actions / Fail |
| Headroom and safety margins | | | Pass / Pass with actions / Fail |
| Load testing and the breaking point | | | Pass / Pass with actions / Fail |
| Scaling mechanisms | | | Pass / Pass with actions / Fail |
| Protection under overload | | | Pass / Pass with actions / Fail |
| Cost and procurement | | | Pass / Pass with actions / Fail |
| Review cadence and ownership | | | Pass / Pass with actions / Fail |

Record the projected exhaustion date for the constraining resource alongside the sign-off, and raise a dated ticket for every item marked "Pass with actions".

## Related checklists

- [Production Readiness Review](/docs/devops/production-readiness/)
- [Observability](/docs/operations/observability/)
- [Cloud Cost Optimization](/docs/cloud/cloud-cost-optimization/)
- [Kubernetes Deployment](/docs/devops/kubernetes-deployment/)
- [Backup and Recovery](/docs/operations/backup-and-recovery/)

## References

- [Google SRE Book — Software Engineering in SRE (Distributed Load Balancing and Capacity)](https://sre.google/sre-book/software-engineering-in-sre/)
- [Google SRE Book — Handling Overload](https://sre.google/sre-book/handling-overload/)
- [Google SRE Book — Addressing Cascading Failures](https://sre.google/sre-book/addressing-cascading-failures/)
- [AWS Well-Architected Framework — Performance Efficiency Pillar](https://docs.aws.amazon.com/wellarchitected/latest/performance-efficiency-pillar/welcome.html)
- [Azure Well-Architected Framework — Performance Efficiency](https://learn.microsoft.com/en-us/azure/well-architected/performance-efficiency/)
