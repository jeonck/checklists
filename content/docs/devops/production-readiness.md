---
title: "Production Readiness Review"
description: "Verify a new service is safe to run in production before it takes real traffic."
icon: "rocket_launch"
weight: 210
toc: true
tags: ["release", "reliability", "sre"]
---

A production readiness review (PRR) is the gate between "it works on staging" and "it carries customer traffic at 3am while nobody is watching". Work through this before the first real request hits the service, and again whenever ownership changes hands.

{{< alert context="info" text="**Who runs this:** the owning team, with one reviewer from outside the team. **When:** at least one week before launch, so findings can actually be fixed." />}}

## 1. Ownership and documentation

- [ ] **A named team owns the service** — not an individual, and the owner is recorded somewhere machine-readable (service catalogue, `CODEOWNERS`, or repo metadata).
- [ ] **The escalation path is written down** — who gets paged first, who gets paged if they do not acknowledge, and who is the business decision-maker for an outage.
- [ ] **A runbook exists and has been read by someone who did not write it** — start-up, shut-down, common failure modes, and how to check whether the service is actually healthy.
- [ ] **The architecture diagram matches reality** — every dependency the service calls, and every caller it serves, is on the diagram.
- [ ] **All upstream and downstream dependencies are listed with their criticality** — mark each as hard (service fails without it) or soft (degrades gracefully).

## 2. Reliability and failure behaviour

- [ ] **Every outbound call has a timeout** — a default-infinite HTTP client is the single most common cause of cascading failure.
- [ ] **Retries use exponential backoff with jitter and a retry budget** — naive retries turn a slow dependency into a self-inflicted denial of service.
- [ ] **Hard dependency failures degrade rather than crash** — decide per dependency whether to fail open, fail closed, or serve stale data, and make it explicit in code.
- [ ] **The service starts cleanly from cold** — no dependency on warm caches, in-memory state from a previous instance, or manual post-start steps.
- [ ] **Health checks distinguish liveness from readiness** — liveness must not fail because a downstream dependency is down, or the orchestrator will restart-loop a healthy process.
- [ ] **Graceful shutdown is implemented** — the process stops accepting new work, drains in-flight requests, and exits within the orchestrator's termination grace period.
- [ ] **Single points of failure are identified and accepted in writing** — a shared database, a single availability zone, or one licence server all count.

{{< alert context="warning" text="**Blocking:** a service without timeouts on outbound calls should not be launched. Everything else on this list can carry a dated follow-up ticket; this one cannot." />}}

## 3. Observability

- [ ] **The four golden signals are on a dashboard** — traffic, error rate, latency (p50/p95/p99), and saturation of the constraining resource.
- [ ] **Logs are structured and include a correlation ID** — free-text logs are unsearchable at the exact moment you need them.
- [ ] **No secrets, tokens, or personal data are written to logs** — grep the codebase for logging of request bodies and auth headers specifically.
- [ ] **Distributed tracing is wired through the request path** — trace context is propagated to every downstream call, not just the first hop.
- [ ] **Log retention is set deliberately** — long enough to investigate a slow-burning bug, short enough to satisfy the data retention policy.
- [ ] **A synthetic probe exercises the critical user journey** — dashboards go green when nobody is using a broken feature.

## 4. Alerting

- [ ] **Alerts fire on user-visible symptoms, not on causes** — "checkout error rate above 2% for 5 minutes" beats "CPU above 80%".
- [ ] **Every alert links to the runbook section that resolves it** — an alert with no documented response is a notification, not an alert.
- [ ] **Alert thresholds have been tested by deliberately breaking something** — in staging, with the on-call rotation watching.
- [ ] **Paging alerts are distinguishable from ticket-generating alerts** — if everything pages, nothing pages.
- [ ] **Alert volume is estimated and is under a page or two per shift** — model it against the last month of staging data if you have nothing else.

## 5. Capacity and performance

- [ ] **A load test has been run against production-like infrastructure** — same instance sizes, same database tier, same network topology.
- [ ] **The breaking point is known** — the request rate at which latency degrades past the SLO, and what fails first when you get there.
- [ ] **Expected launch traffic is documented with a peak-to-average ratio** — including any marketing spike, batch job, or scheduled import.
- [ ] **Autoscaling limits are set and the maximum is affordable** — check the maximum against the monthly budget, not just the technical limit.
- [ ] **Resource requests and limits are set from measured usage** — not copied from another service's manifest.
- [ ] **Rate limiting or load shedding protects the service from a bad client** — including your own retrying clients.

## 6. Data and state

- [ ] **Backups are configured and a restore has actually been performed** — an untested backup is a hypothesis.
- [ ] **The recovery time and recovery point objectives are written down and achievable** — measure the restore, do not estimate it.
- [ ] **Schema migrations are backward compatible** — the previous version of the application must run against the new schema for the duration of the rollout.
- [ ] **Data at rest and in transit is encrypted** — including backups, snapshots, and any object storage bucket.
- [ ] **Personal data is classified and its retention period is enforced by a job, not by intention.**

## 7. Security

- [ ] **The service authenticates its callers and authorises every request** — network position is not authorisation.
- [ ] **Secrets come from a secret manager at runtime** — not from environment variables baked into an image, and never from the repository.
- [ ] **Dependencies have been scanned and known critical vulnerabilities are resolved or accepted in writing.**
- [ ] **The container runs as a non-root user with a read-only root filesystem where possible.**
- [ ] **Network access is least-privilege** — the service can reach only the dependencies it declared in section 1.
- [ ] **An audit log records security-relevant events** — authentication, authorisation failures, and privileged actions.

## 8. Release and rollback

- [ ] **Deployment is fully automated and repeatable** — the same command, run by anyone on the team, from a clean checkout.
- [ ] **Rollback has been rehearsed and is a single documented action** — and it works even when the deployment pipeline is broken.
- [ ] **The rollout is progressive** — canary, blue/green, or percentage-based, with automated abort on error-rate regression.
- [ ] **Feature flags gate risky behaviour and are removed on a schedule** — a permanent flag is a permanent untested code path.
- [ ] **Rollback is safe with respect to data** — if the new version wrote data the old version cannot read, rollback is a lie.

## 9. Cost

- [ ] **The expected monthly cost is estimated and has an owner.**
- [ ] **Every resource is tagged for cost allocation** — service, team, and environment at minimum.
- [ ] **A budget alert is configured at a threshold that gives you time to react.**

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Reliability and failure behaviour | | | Pass / Pass with actions / Fail |
| Observability and alerting | | | Pass / Pass with actions / Fail |
| Capacity and performance | | | Pass / Pass with actions / Fail |
| Data, backup, and recovery | | | Pass / Pass with actions / Fail |
| Security | | | Pass / Pass with actions / Fail |
| Release and rollback | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with an owner before the launch is approved.

## Related checklists

- [Kubernetes Deployment](/docs/devops/kubernetes-deployment/)
- [Observability](/docs/operations/observability/)
- [Backup and Recovery](/docs/operations/backup-and-recovery/)
- [On-Call Handover](/docs/operations/on-call-handover/)

## References

- [Google SRE Workbook — Implementing SLOs](https://sre.google/workbook/implementing-slos/)
- [Google SRE Book — The Production Readiness Review](https://sre.google/sre-book/evolving-sre-engagement-model/)
- [AWS Well-Architected Framework — Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)
