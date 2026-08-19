---
title: "Cloud Cost Optimisation"
description: "Verify cloud spend is attributed, free of waste, and covered by the right commitments before you cut budgets."
icon: "savings"
weight: 320
toc: true
tags: ["finops", "cost", "cloud", "optimisation"]
---

Most cloud cost programmes fail in the same way: someone is asked to cut 30%, they turn off a few instances, and six months later spend is back where it started. Durable savings come from attribution first, then waste removal, then rightsizing, then commitments — in that order, because buying a three-year commitment for an oversized fleet locks in the waste. Work through this in sequence rather than jumping to the discount instruments.

{{< alert context="info" text="**Who runs this:** a FinOps practitioner or platform engineer, paired with the engineering owner of each major workload. **When:** quarterly as a standing review, and before any commitment purchase or budget-cut exercise." />}}

## 1. Visibility and cost attribution {#visibility-and-cost-attribution}

- [ ] **The detailed billing export is enabled and queryable** — AWS Cost and Usage Report into Athena, Azure cost export into a storage account, or GCP billing export into BigQuery; the console cost explorer is too coarse to answer per-team questions.
- [ ] **At least 90% of spend is attributable to a team or product** — measure the unattributed percentage explicitly, because that number is the ceiling on any accountability you try to introduce.
- [ ] **Shared costs have a documented, agreed allocation rule** — network egress, the Kubernetes control plane, observability tooling; unallocated shared cost is where accountability quietly disappears.
- [ ] **Tag coverage is measured and reported, not assumed** — run a query for untagged resource cost by service and treat the result as a defect backlog.
- [ ] **Kubernetes spend is broken down per namespace or workload** — a cluster shows up as one EC2 line item, which means the twenty teams sharing it all see zero cost.
- [ ] **Each team can see their own spend without asking finance** — a monthly emailed spreadsheet produces no behaviour change; a dashboard the team owns does.
- [ ] **A unit cost metric exists for the main business driver** — cost per order, per active user, per GB ingested; absolute spend rising while unit cost falls is a healthy business, and you cannot tell the difference without the ratio.
- [ ] **Cost anomaly detection is enabled and alerts reach the owning team, not just finance.**

## 2. Eliminating waste {#eliminating-waste}

- [ ] **Unattached block storage volumes are identified and deleted** — orphaned EBS, managed disks, and persistent disks survive instance termination and accrue indefinitely with zero signal.
- [ ] **Old snapshots and machine images beyond the retention policy are removed** — snapshot sprawl is incremental per snapshot but the chain keeps the original blocks alive forever.
- [ ] **Unassociated static IP addresses are released** — an idle elastic IP is billed precisely because it is idle.
- [ ] **Idle load balancers and NAT gateways are removed** — both bill hourly regardless of traffic, and a NAT gateway left in a decommissioned VPC is pure loss.
- [ ] **Compute instances with sustained near-zero CPU and network are challenged with their owner** — set a threshold and a lookback window (for example under 3% CPU and under 5 MB/day network over 30 days) rather than eyeballing a graph.
- [ ] **Non-production environments are scheduled off outside working hours** — a development fleet running 168 hours a week instead of 50 wastes roughly 70% of its cost, and this is usually the single largest quick win.
- [ ] **Provisioned-capacity databases and search clusters in non-production are downsized or switched to serverless tiers.**
- [ ] **Abandoned resources from proofs of concept have an expiry mechanism** — sandbox accounts that reap resources on a timer, or a tag with a deletion date that a scheduled job enforces.
- [ ] **Data transfer to deleted or unused observability and log destinations is stopped** — teams frequently delete a dashboard and leave the agent shipping the data.

## 3. Rightsizing compute {#rightsizing-compute}

- [ ] **Rightsizing recommendations are reviewed against a lookback window of at least 14 days including peak** — a recommendation generated over a quiet fortnight will undersize a workload that spikes monthly.
- [ ] **Memory utilisation is actually being collected** — provider rightsizing defaults to CPU only, and memory-bound workloads get downsized into an out-of-memory incident.
- [ ] **Instance families match the workload shape** — moving a memory-bound service off a general-purpose family onto a memory-optimised one is usually cheaper per unit of useful capacity, not more expensive.
- [ ] **Current-generation and ARM-based instance types have been evaluated** — Graviton, Ampere Altra, and Azure Cobalt typically offer a materially better price-performance ratio for workloads that recompile cleanly.
- [ ] **Kubernetes requests are set from observed usage percentiles, not copied from another manifest** — the cluster bills for requests, so an over-requested pod wastes capacity even while sitting idle.
- [ ] **Cluster autoscaling and bin-packing are configured, with a measured node utilisation target** — a cluster averaging 25% allocation is paying four times over.
- [ ] **Autoscaling is driven by the metric that actually constrains the service** — scaling on CPU for a queue-depth-bound consumer means over-provisioning while the backlog grows.
- [ ] **Rightsizing changes are rolled out progressively and monitored against latency SLOs** — a saving that causes an incident is not a saving.

## 4. Commitments and discount instruments {#commitments-and-discount-instruments}

- [ ] **Waste removal and rightsizing were completed before commitments were sized** — this is the ordering mistake that locks in years of overspend.
- [ ] **Commitment coverage and utilisation are both tracked** — coverage tells you how much on-demand spend is still uncovered; utilisation tells you whether you are paying for commitments you do not consume, and only both together are meaningful.
- [ ] **Commitments are sized to the stable baseline, not the peak** — target covering roughly the trough of the last 90 days and let on-demand or spot absorb the variable layer.
- [ ] **The commitment mix balances flexibility against discount depth** — compute savings plans and flexible reservations cost a little more than instance-locked ones and are worth it if your instance types will change within the term.
- [ ] **Commitments are purchased from the management or billing account so discounts share across the estate** — a reservation stranded in one account benefits nobody else.
- [ ] **Expiry dates are calendared with an owner and a renewal decision** — a lapsed commitment silently reverts an entire fleet to on-demand pricing.
- [ ] **Spot or preemptible capacity is used for fault-tolerant workloads** — batch, CI runners, and stateless consumers; with diversified instance pools and interruption handling actually implemented and tested.
- [ ] **Committed-use discounts on managed services are evaluated too** — databases, caches, and data warehouses often have separate reservation programmes that get overlooked.

{{< alert context="warning" text="**Common mistake:** buying a three-year commitment to hit a quarterly savings target before rightsizing. You will be paying for oversized instances until 2029, and the commitment cannot be undone — only partially resold, in some regions, at a loss." />}}

## 5. Storage and data lifecycle {#storage-and-data-lifecycle}

- [ ] **Every object storage bucket over 100 GB has a lifecycle policy** — and the transition thresholds were derived from access log analysis rather than guessed.
- [ ] **Storage class analysis has been run before enabling intelligent tiering everywhere** — the monitoring charge per object makes automatic tiering a loss for buckets full of small objects.
- [ ] **Incomplete multipart uploads are aborted by a lifecycle rule** — these are invisible in the console object listing and bill indefinitely; almost every mature estate has some.
- [ ] **Versioned buckets expire non-current versions** — versioning without expiry means storage grows without bound for data that is being actively overwritten.
- [ ] **Log and metric retention is set per stream according to how far back anyone has actually looked** — the default of indefinite retention on high-cardinality telemetry is often a top-five line item.
- [ ] **Block storage volume types match their IO profile** — gp3 or equivalent decouples IOPS from capacity, so provisioned-IOPS volumes should be reserved for workloads that measurably need them.
- [ ] **Backup retention matches the stated recovery point objective and legal requirement, not a default** — and duplicate backups from two overlapping tools have been eliminated.
- [ ] **Database storage auto-growth is monitored** — many managed databases grow storage automatically and never shrink it, so a one-off import permanently raises the floor.

## 6. Network and data transfer {#network-and-data-transfer}

- [ ] **Data transfer charges are broken out and understood by direction and boundary** — cross-AZ, cross-region, internet egress, and NAT processing are separately priced and separately fixable.
- [ ] **Cross-availability-zone chatter is measured** — a service mesh or a naive client that ignores topology can double a bill by routing every request across zones for no availability benefit.
- [ ] **VPC endpoints replace NAT gateway paths for high-volume provider services** — S3 and DynamoDB gateway endpoints have no data processing charge.
- [ ] **A CDN fronts high-volume public egress** — CDN egress is cheaper than origin egress and reduces origin compute at the same time.
- [ ] **Cache hit ratio at the CDN is measured and tuned** — a misconfigured cache key or a missing cache-control header turns a CDN into an expensive proxy.
- [ ] **Chatty cross-region replication is justified by an actual availability requirement** — replicating everything to a DR region that has never been used costs storage plus transfer plus the compute to serve it.

## 7. Managed services, licensing, and vendors {#managed-services-licensing-and-vendors}

- [ ] **Serverless versus provisioned has been compared on real traffic shape** — spiky, low-duty-cycle workloads favour serverless; steady high-utilisation workloads are usually cheaper provisioned, and the crossover is workload-specific.
- [ ] **Provisioned throughput on databases, streams, and search matches measured demand** — provisioned capacity on a table with bursty traffic is worse than on-demand mode on both cost and reliability.
- [ ] **Licence-included instances are compared against bring-your-own-licence** — Windows and commercial database licensing frequently exceeds the compute cost it rides on.
- [ ] **Third-party SaaS billed on host, node, or ingest volume is reviewed alongside cloud spend** — observability and security tooling commonly scales with your infrastructure and is not in the cloud bill at all.
- [ ] **Duplicate tooling doing the same job has been consolidated** — two log platforms, three secrets stores, and a CI system nobody uses is a recurring pattern after any reorganisation or acquisition.
- [ ] **Enterprise discount programme and marketplace private offers have been negotiated at renewal** — committed spend agreements and marketplace routing of third-party spend both move real money.

## 8. Governance and operating model {#governance-and-operating-model}

- [ ] **Cost is a visible engineering metric, not a finance report** — put unit cost on the same dashboard as latency and error rate, and it becomes an engineering concern.
- [ ] **New workloads produce a cost estimate before provisioning** — infrastructure-as-code cost estimation in pull requests catches the expensive default before it is merged.
- [ ] **Budgets exist per team with alerts routed to that team's channel** — an alert to a central inbox generates no behaviour change.
- [ ] **A savings backlog exists with estimated value, effort, and owner** — treat it as a product backlog, or it stays a spreadsheet.
- [ ] **Realised savings are tracked against forecast** — most reported cloud savings never appear in the bill because the resource came back under a different name.
- [ ] **Guardrails prevent the highest-cost mistakes** — instance type allow-lists, region restrictions, and a hard cap on GPU families in non-production.
- [ ] **A review cadence is scheduled with named attendees** — cost optimisation is a rate, not a project, and the estate regrows waste continuously.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Visibility and cost attribution | | | Pass / Pass with actions / Fail |
| Eliminating waste | | | Pass / Pass with actions / Fail |
| Rightsizing compute | | | Pass / Pass with actions / Fail |
| Commitments and discount instruments | | | Pass / Pass with actions / Fail |
| Storage and data lifecycle | | | Pass / Pass with actions / Fail |
| Network and data transfer | | | Pass / Pass with actions / Fail |
| Managed services, licensing, and vendors | | | Pass / Pass with actions / Fail |
| Governance and operating model | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated item on the savings backlog with an owner and an estimated value.

## Related checklists

- [Cloud Landing Zone Setup](/docs/cloud/aws-landing-zone/)
- [Capacity Planning](/docs/operations/capacity-planning/)
- [Serverless Readiness](/docs/cloud/serverless-readiness/)
- [Kubernetes Deployment](/docs/devops/kubernetes-deployment/)
- [Observability](/docs/operations/observability/)

## References

- [FinOps Foundation — FinOps Framework](https://www.finops.org/framework/)
- [AWS Well-Architected Framework — Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html)
- [Microsoft Azure Well-Architected Framework — Cost Optimization](https://learn.microsoft.com/en-us/azure/well-architected/cost-optimization/)
- [Google Cloud — Cost optimisation on Google Cloud](https://cloud.google.com/architecture/framework/cost-optimization)
