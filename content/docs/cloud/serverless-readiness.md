---
title: "Serverless Readiness"
description: "Verify a serverless workload handles scale, retries, cold starts, and failure before it takes production traffic."
icon: "bolt"
weight: 340
toc: true
tags: ["serverless", "lambda", "event-driven", "reliability"]
---

Serverless removes the servers, not the distributed systems problems. What it changes is which problems bite you: concurrency limits instead of instance counts, at-least-once delivery instead of a request-response call, and a cost model where an infinite retry loop bills by the millisecond. This checklist covers function-as-a-service workloads on AWS Lambda, Azure Functions, and Google Cloud Run functions, along with the event sources that trigger them.

{{< alert context="info" text="**Who runs this:** the owning engineering team, with a reviewer who has operated an event-driven system before. **When:** before the workload takes production traffic, and again after any change to its event sources or concurrency configuration." />}}

## 1. Workload fit

- [ ] **The workload's execution profile fits within the platform's timeout and payload limits** — with headroom, because a job that takes 12 minutes against a 15-minute ceiling will time out the first time a dependency is slow.
- [ ] **Long-running or steady high-throughput work has been compared against containers on cost and latency** — serverless wins on spiky and low-duty-cycle workloads; a service at constant high utilisation is usually cheaper and faster on provisioned compute.
- [ ] **The function does one thing** — a single function fronting a dozen routes with a switch statement inherits the union of every route's permissions, memory profile, and blast radius.
- [ ] **Orchestration of multi-step workflows uses a workflow service, not chained function invocations** — Step Functions, Durable Functions, or Workflows give you state, retries, and visibility that a function calling a function does not.
- [ ] **State lives outside the function** — anything cached in the execution environment survives only until the platform recycles it, which it will do without warning and at the worst time.
- [ ] **Dependencies that hold connection pools have a strategy** — a relational database behind a function that scales to a thousand concurrent executions needs a proxy or a data API, or it will exhaust connections.

## 2. Configuration and packaging

- [ ] **Memory is tuned by measurement, not left at the default** — CPU is allocated proportionally to memory, so more memory often reduces both duration and total cost; run a power-tuning sweep rather than guessing.
- [ ] **Timeout is set to a realistic ceiling per function, not the platform maximum** — a long timeout means a stuck invocation bills for minutes and holds a concurrency slot the whole time.
- [ ] **The deployment package excludes development dependencies, tests, and the provider SDK where the runtime supplies it** — package size directly affects cold start.
- [ ] **Shared code is in a layer or an internal package with a pinned version** — a layer that changes underneath deployed functions is an untracked production change.
- [ ] **The runtime version is current and there is a plan for its deprecation date** — providers force-migrate deprecated runtimes on a published schedule, and being surprised by it is avoidable.
- [ ] **Configuration comes from environment variables or a parameter store, and secrets from a secret manager** — with the fetch cached outside the handler so it does not run on every invocation.
- [ ] **Ephemeral storage size and usage are known** — the writable temporary directory is shared across invocations in the same environment and does not empty itself.

## 3. Cold starts and latency

- [ ] **Cold start latency has been measured, not assumed** — measure the p99 of the initialisation phase separately from execution, because averages hide it entirely.
- [ ] **Initialisation work happens outside the handler** — SDK clients, database connections, and configuration loading belong in the module scope so they are reused across warm invocations.
- [ ] **The cold start contribution to the user-facing latency SLO is quantified** — for an asynchronous consumer it may be irrelevant; for a synchronous API path it may be the dominant term.
- [ ] **Provisioned or minimum-instance concurrency is configured where cold start would breach the SLO** — and its cost is understood, since it bills whether or not it is used.
- [ ] **Functions in a VPC have been checked for the network initialisation penalty and only use a VPC when they need private resources.**
- [ ] **Ahead-of-time compilation or a lighter runtime has been considered for latency-critical paths** — startup-optimised runtimes and native images cut initialisation by an order of magnitude for JVM and .NET workloads.

## 4. Concurrency, scaling, and downstream protection

- [ ] **The account-level concurrency limit is known and the headroom across all functions is calculated** — concurrency is a shared account quota, so one runaway function can starve every other function in the account.
- [ ] **Reserved concurrency is set on critical functions to guarantee capacity** — and on risky ones to cap their blast radius.
- [ ] **Downstream capacity has been checked against the function's maximum concurrency** — a function scaling to 1,000 concurrent executions against a database that accepts 100 connections is a self-inflicted outage.
- [ ] **Burst scaling behaviour is understood for the region** — platforms scale in bursts then at a rate limit, so a traffic spike produces throttling before it produces capacity.
- [ ] **Throttling is monitored and alerted on** — a throttled synchronous invocation is a user-facing error; a throttled asynchronous one is a delayed or eventually dropped event.
- [ ] **Batch size and batching window for stream and queue sources are tuned deliberately** — larger batches improve throughput and cost but increase the amount of work lost or retried on a single failure.
- [ ] **A load test has driven the function to its concurrency ceiling** — the interesting failures all happen at the limit, not below it.

## 5. Event sources, delivery, and idempotency

- [ ] **Delivery semantics are known per event source and written down** — at-least-once for most queue and stream sources means duplicate invocations are normal operation, not an incident.
- [ ] **Every handler with a side effect is idempotent** — deduplicate on an event identifier in a store with a TTL, or make the write naturally idempotent; retries will otherwise double-charge a customer.
- [ ] **Ordering assumptions are explicit** — only partitioned stream sources preserve order, and only within a partition; standard queues do not.
- [ ] **Partition or shard key choice distributes load evenly** — a hot key serialises an entire workload behind one consumer regardless of how much the platform scales.
- [ ] **Event schemas are versioned and consumers tolerate unknown fields** — a producer adding a field should never break a consumer.
- [ ] **Queue visibility timeout is at least the function timeout, usually a multiple of it** — a shorter visibility timeout means the message is redelivered while the first invocation is still processing it.
- [ ] **Stream consumers handle a poison record** — without bisect-on-error or a failure destination, one unparseable record blocks its shard until the retention period expires.
- [ ] **Fan-out patterns are checked for accidental recursion** — a function writing to the bucket that triggers it is the classic infinite loop, and it bills continuously until someone notices.

{{< alert context="danger" text="**Blocking:** a function that writes to its own trigger source without a guard is a recursive invocation loop. Verify the write path cannot re-trigger the function, and set reserved concurrency as a hard ceiling before deploying." />}}

## 6. Failure handling

- [ ] **Every asynchronous invocation path has a dead letter queue or an on-failure destination** — without one, an event that fails all retries is silently discarded and you will never know it existed.
- [ ] **The dead letter queue has an alarm on depth greater than zero and a documented owner** — an unmonitored DLQ is a data loss mechanism with extra steps.
- [ ] **A tested procedure exists for inspecting and replaying dead-lettered events** — replay is not automatic and writing the tooling during an incident is too late.
- [ ] **Retry counts and backoff are configured deliberately per source** — the platform defaults are rarely right, and retrying a permanent validation failure twice is wasted spend plus delay.
- [ ] **Transient and permanent errors are distinguished in code** — a malformed payload should go straight to the DLQ; a downstream timeout should retry.
- [ ] **Partial batch failure reporting is implemented for batched sources** — otherwise one bad record in a batch of a hundred causes all hundred to be reprocessed.
- [ ] **Downstream calls have explicit timeouts shorter than the function timeout** — a default-infinite SDK client will burn the entire timeout and then fail with no useful error.
- [ ] **Circuit-breaking or shedding protects a failing downstream** — thousands of concurrent executions all retrying a struggling dependency will keep it down.

## 7. Security

- [ ] **Each function has its own execution role scoped to the specific resources it uses** — a shared role across functions grants every function the union of all permissions.
- [ ] **Wildcard resource ARNs have been eliminated or justified** — `Resource: "*"` on a data action is the finding every audit will raise.
- [ ] **Resource policies restrict who can invoke the function** — the invoker principal, source ARN, and source account should all be constrained.
- [ ] **Input from event sources is validated and treated as untrusted** — an event body from a queue is user input that has travelled through one more hop.
- [ ] **Secrets are fetched at runtime from a secret manager, cached with a bounded TTL, and never logged** — environment variables are visible to anyone with read access to the function configuration.
- [ ] **Dependencies are scanned in the pipeline and the package is rebuilt on a schedule** — a function deployed a year ago is still running a year-old dependency tree.
- [ ] **Functions in a VPC use security groups that permit only the required egress** — and a function that needs no private resource stays out of the VPC.
- [ ] **API-fronted functions have authentication, authorisation, and rate limiting at the gateway** — the gateway is the only place you can reject traffic before it costs you an invocation.

## 8. Observability

- [ ] **Structured JSON logging is used with a correlation identifier propagated from the event** — free-text logs across hundreds of short-lived invocations are unsearchable.
- [ ] **Log retention is set explicitly on every log group** — the default in several platforms is to never expire, and high-volume function logs become a significant line item.
- [ ] **Distributed tracing is enabled and covers the asynchronous hops** — an event-driven system where the trace stops at the queue tells you nothing about end-to-end latency.
- [ ] **Alarms cover errors, throttles, duration approaching timeout, and DLQ depth** — duration approaching timeout is the leading indicator that catches the failure before it happens.
- [ ] **Business-level metrics are emitted from the function** — events processed, events rejected, and end-to-end age of the oldest processed event.
- [ ] **Iterator age or queue age is alerted on for stream and queue consumers** — a consumer keeping up at a steadily growing lag is the failure mode invisible to error-rate alerts.
- [ ] **A method exists to reproduce an invocation locally from a captured event payload** — debugging by redeploying is unbearably slow.

## 9. Deployment, cost, and operations

- [ ] **Infrastructure and functions are deployed together as code from a pipeline** — a function edited in the console is a change nobody can review or reproduce.
- [ ] **Deployments are progressive with automatic rollback on an error-rate alarm** — weighted aliases or traffic-splitting revisions make this cheap, so there is no reason not to.
- [ ] **Function versions are immutable and aliases point at a specific version** — deploying to a mutable latest pointer removes your ability to roll back precisely.
- [ ] **Cost per million invocations has been estimated including the event sources and log ingestion** — the function is often the cheapest part of the bill; the queue, gateway, and logs are not.
- [ ] **A budget alarm and a concurrency ceiling limit the cost of a runaway loop** — serverless removes the natural ceiling that a fixed fleet used to provide.
- [ ] **A runbook covers how to stop the workload** — setting reserved concurrency to zero or disabling the event source mapping is the emergency brake, and the on-call engineer must know which one applies.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Workload fit | | | Pass / Pass with actions / Fail |
| Configuration and packaging | | | Pass / Pass with actions / Fail |
| Cold starts and latency | | | Pass / Pass with actions / Fail |
| Concurrency, scaling, and downstream protection | | | Pass / Pass with actions / Fail |
| Event sources, delivery, and idempotency | | | Pass / Pass with actions / Fail |
| Failure handling | | | Pass / Pass with actions / Fail |
| Security | | | Pass / Pass with actions / Fail |
| Observability | | | Pass / Pass with actions / Fail |
| Deployment, cost, and operations | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with an owner before the workload is approved for production traffic.

## Related checklists

- [Production Readiness Review](/docs/devops/production-readiness/)
- [Observability](/docs/operations/observability/)
- [Cloud Cost Optimisation](/docs/cloud/cloud-cost-optimization/)
- [REST API Design](/docs/development/rest-api-design/)
- [Secrets Management](/docs/security/secrets-management/)

## References

- [AWS Lambda Developer Guide — Best practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [AWS Well-Architected Framework — Serverless Applications Lens](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/welcome.html)
- [Azure Functions — Best practices for reliable Azure Functions](https://learn.microsoft.com/en-us/azure/azure-functions/functions-best-practices)
- [Google Cloud Run functions documentation](https://cloud.google.com/functions/docs)
