---
title: "Observability"
description: "Verify a service emits the signals needed to diagnose an unfamiliar failure without shipping new code."
icon: "insights"
weight: 510
toc: true
tags: ["observability", "monitoring", "sre", "telemetry"]
---

Monitoring answers questions you thought of in advance. Observability is whether you can answer a question you have never asked before, at 3am, without deploying anything. This checklist covers the instrumentation, the pipeline that carries it, and the dashboards and alerts built on top — in the order you would build them.

{{< alert context="info" text="**Who runs this:** the owning team, reviewed by whoever is on call for the service. **When:** before a production readiness review, and again after any incident where the answer was *we could not tell*." />}}

## 1. Signal coverage

- [ ] **The four golden signals exist for every user-facing entry point** — traffic, error rate, latency distribution, and saturation of the constraining resource, per endpoint rather than only in aggregate.
- [ ] **Latency is recorded as a histogram, not an average** — averages hide the tail, and the tail is what your users are complaining about.
- [ ] **Errors are split by cause** — client error, dependency failure, timeout, and unhandled exception behave differently and need different responses.
- [ ] **Queue depth and consumer lag are instrumented for every asynchronous path** — an async system fails silently by falling behind, not by returning errors.
- [ ] **Business-level counters exist alongside technical ones** — orders placed, sign-ups completed, payments settled; a drop here is real even when every technical metric looks green.
- [ ] **Batch and scheduled jobs emit start, finish, duration, and record counts** — a cron job that stopped running produces no signal at all unless you explicitly watch for its absence.
- [ ] **Dependency calls are instrumented on the client side** — a downstream service that reports itself healthy can still be unreachable from your network position.

## 2. Metrics

- [ ] **Metric names follow one documented convention** — unit suffix, consistent prefix, and the same label spelling across services, or cross-service queries silently return nothing.
- [ ] **Cardinality is bounded and reviewed** — never label a metric with user ID, request ID, URL path with embedded IDs, or anything unbounded; this is the single most common way to take down a metrics backend.
- [ ] **Counters are monotonic and reset-safe** — use rate functions over counters rather than gauges that your code decrements, so a restart does not produce a negative spike.
- [ ] **Scrape or push interval is short enough to see the incidents you care about** — a 60-second interval cannot show you a 30-second outage.
- [ ] **Recording rules pre-compute the expensive queries used by dashboards and alerts** — a dashboard that takes 40 seconds to load will not be opened during an incident.
- [ ] **Retention is tiered deliberately** — high resolution for recent debugging, downsampled long-term series for capacity and trend work.

## 3. Logs

- [ ] **All logs are structured** — JSON or an equivalent key-value format, so that fields can be filtered and aggregated rather than grepped.
- [ ] **Every log line carries a trace ID and a request or job ID** — without a correlation key, a distributed request becomes a hundred unlinked lines across a dozen services.
- [ ] **Log levels are used consistently and documented** — if everything is logged at ERROR, error-rate panels and log-based alerts are worthless.
- [ ] **No secrets, credentials, tokens, or personal data reach the log pipeline** — audit request-body logging and auth-header logging specifically, and add a redaction filter at the collector as a backstop.
- [ ] **Log volume per request is bounded** — a debug-level loop inside a hot path can cost more than the service itself and will be discovered on the invoice.
- [ ] **Retention satisfies both the investigation need and the data retention policy** — long enough to chase a slow-burning bug, short enough to be defensible.
- [ ] **Log ingestion failure is itself monitored** — a full disk or a rejected batch means you are flying blind precisely when things are going wrong.

## 4. Distributed tracing

- [ ] **Trace context propagates across every hop** — HTTP, gRPC, message queues, and background workers; a trace that stops at the first hop tells you nothing you did not already know.
- [ ] **Spans are created for outbound calls, database queries, and cache lookups** — the point of tracing is attributing latency to a component, which needs spans at component boundaries.
- [ ] **Sampling is head-based with a tail-based override for errors and slow requests** — uniform 1% sampling reliably discards the exact traces you want.
- [ ] **Span attributes include the things you would filter on** — tenant, region, endpoint, and version, without embedding unbounded values.
- [ ] **Instrumentation uses OpenTelemetry or another vendor-neutral SDK** — so that changing backend does not mean re-instrumenting every service.
- [ ] **Trace IDs are surfaced to users or in error responses** — a support ticket that quotes a trace ID turns a two-hour investigation into a two-minute one.

## 5. Dashboards

- [ ] **Every service has one overview dashboard that fits on a single screen** — golden signals plus dependency health, with no scrolling required.
- [ ] **Dashboards are ordered top-down** — user-visible symptoms first, then service internals, then infrastructure, matching how you actually diagnose.
- [ ] **Each panel has a stated purpose and a known-good range** — a graph nobody can interpret under pressure is decoration.
- [ ] **Deploy and configuration-change markers are overlaid on the time axis** — most incidents correlate with a change, and this makes the correlation instant.
- [ ] **Dashboards are defined as code and version-controlled** — a hand-edited dashboard is lost the first time someone else edits it.
- [ ] **A cross-service view exists for the critical user journey** — end-to-end latency and success rate for the journey, not just for individual services.

## 6. Alerting

- [ ] **Alerts fire on user-visible symptoms, not on causes** — high CPU is only worth paging about if it is degrading something a user can perceive.
- [ ] **Every alert that pages links to the runbook section that resolves it** — an alert with no documented response should be downgraded to a ticket.
- [ ] **Paging alerts and ticketing alerts are separate channels with separate severities** — if everything pages, nothing pages.
- [ ] **Alert conditions include a duration so transient blips do not page** — and the duration is short enough that the alert still fires inside the error budget window.
- [ ] **Absence of data triggers an alert** — a crashed exporter looks exactly like a perfectly healthy system on most dashboards.
- [ ] **Alert rules are tested by deliberately breaking something in staging** — an untested alert rule is a hypothesis about a query language.
- [ ] **Alert volume per shift is measured and reviewed monthly** — track the noisiest alerts and either fix, retune, or delete them.

{{< alert context="warning" text="**Common mistake:** alerting on every metric that has a threshold. Each new page must be justified against the question *what would the responder do differently in the next five minutes?* If there is no answer, it is a dashboard panel, not an alert." />}}

## 7. SLOs and error budgets

- [ ] **Each critical user journey has an SLI defined as good events divided by valid events** — with an explicit definition of what counts as good and what requests are excluded.
- [ ] **SLO targets are set from what users need, not from what the system currently does** — and they are achievable, since an SLO that is permanently breached is ignored.
- [ ] **The error budget and its burn rate are visible on the service dashboard** — a burn-rate alert catches fast-burning incidents far earlier than a raw threshold.
- [ ] **Multi-window multi-burn-rate alerting is configured** — a fast window for outages and a slow window for chronic degradation, so both are caught without false pages.
- [ ] **A policy exists for what happens when the budget is exhausted** — typically freezing feature releases in favour of reliability work, agreed with the product owner in advance.

## 8. Synthetic monitoring and the pipeline itself

- [ ] **A synthetic probe exercises the critical user journey from outside the network** — real user traffic can drop to zero while every internal metric stays green.
- [ ] **Probes run from more than one region or provider** — so that a probe failure is distinguishable from an outage in the probing infrastructure.
- [ ] **The telemetry pipeline has its own monitoring and its own alerting path** — collector health, drop rates, and backend ingestion latency, alerted through a channel that does not depend on the pipeline.
- [ ] **Observability cost is attributed per service and reviewed** — telemetry regularly becomes a top-three line item and is best trimmed before finance asks.
- [ ] **Data survives the loss of the primary region** — if the observability stack lives only in the region that just failed, you will debug the outage from memory.
- [ ] **On-call engineers have read access to every signal without raising a request** — an access-request queue during an incident is an outage extender.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Signal coverage | | | Pass / Pass with actions / Fail |
| Metrics | | | Pass / Pass with actions / Fail |
| Logs | | | Pass / Pass with actions / Fail |
| Distributed tracing | | | Pass / Pass with actions / Fail |
| Dashboards | | | Pass / Pass with actions / Fail |
| Alerting | | | Pass / Pass with actions / Fail |
| SLOs and error budgets | | | Pass / Pass with actions / Fail |
| Synthetic monitoring and the pipeline itself | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with a named owner, and re-review any area marked Fail before the service is treated as observable.

## Related checklists

- [Production Readiness Review](/docs/devops/production-readiness/)
- [On-Call Handover](/docs/operations/on-call-handover/)
- [Incident Management](/docs/operations/incident-management/)
- [Capacity Planning](/docs/operations/capacity-planning/)
- [Kubernetes Deployment](/docs/devops/kubernetes-deployment/)

## References

- [Google SRE Book — Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/)
- [Google SRE Workbook — Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
- [Prometheus — Instrumentation and Naming Best Practices](https://prometheus.io/docs/practices/naming/)
- [AWS Well-Architected Framework — Operational Excellence Pillar](https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/welcome.html)
