---
title: "Release Day Runbook"
description: "A time-ordered runbook for taking a release from final preparation through deploy, soak, and rollback decision."
icon: "event_available"
weight: 260
toc: true
tags: ["release", "deployment", "runbook", "on-call"]
---

Release day goes badly when decisions that should have been made yesterday are made under time pressure with an audience watching. This runbook is ordered by time rather than by topic: work down it from the day before the release to the point where the release is declared done. Assign a single release conductor who owns the sequence and the go/no-go call.

{{< alert context="info" text="**Who runs this:** a named release conductor, with the owning engineers and the on-call responder present. **When:** starting the day before the deploy window and continuing through the soak period." />}}

## 1. T-1 day: scope and readiness {#t-1-day-scope-and-readiness}

- [ ] **The release contents are frozen and the change list is published** — every merged commit, with the pull requests and tickets it closes, so nobody discovers an unexpected change mid-incident.
- [ ] **Every change in the release has been through code review and the pipeline is green on the release commit** — a green build from an earlier commit does not count.
- [ ] **The exact artefact digest to be deployed is recorded** — not a tag, so the thing you tested and the thing you ship are provably identical.
- [ ] **The release has been running in a staging environment with production-like data and traffic for long enough to expose slow failures** — memory leaks and connection pool exhaustion do not appear in a ten-minute smoke test.
- [ ] **Risky changes are behind feature flags that default to off** — this converts a deploy decision into a separate, reversible enablement decision.
- [ ] **Database migrations are backward compatible and have been timed against a production-sized dataset** — a migration that locks a large table for minutes is an outage regardless of how correct it is.
- [ ] **Any dependency on another team's release is confirmed in writing, including its ordering.**

## 2. T-1 day: communication and logistics {#t-1-day-communication-and-logistics}

- [ ] **The deploy window is agreed and published** — with the timezone stated explicitly and confirmed against any change freeze.
- [ ] **The window avoids peak traffic, the end of a shift, and the hours before a weekend or holiday** — deploying at 5pm on a Friday leaves nobody to watch the soak.
- [ ] **Named people are assigned to each role** — release conductor, deployer, verifier, and the on-call responder who will own anything that goes wrong afterwards.
- [ ] **Everyone in those roles has confirmed availability for the deploy window plus the soak period.**
- [ ] **Support, customer-facing, and dependent teams have been notified with what user-visible change to expect** — including anything they should not raise as a bug.
- [ ] **The incident channel and bridge are created in advance** — creating communication channels during an incident wastes the first ten minutes.
- [ ] **Anything requiring a maintenance window or a status page notice has been scheduled and drafted.**

## 3. T-1 day: rollback preparation {#t-1-day-rollback-preparation}

- [ ] **The rollback procedure is written down as concrete commands, not as a description** — the person running it may not be the person who wrote it.
- [ ] **The previous artefact version and its digest are recorded and confirmed to still exist in the registry** — retention policies have deleted rollback targets before.
- [ ] **Rollback has been rehearsed in staging within the last release cycle** — an unrehearsed rollback is a hypothesis.
- [ ] **The data implications of a rollback are understood** — if the new version writes data or schema the old version cannot read, rollback is not actually available and you need a forward-fix plan instead.
- [ ] **The rollback trigger conditions are written down in advance** — specific thresholds such as error rate above a stated value for a stated duration, not a judgement call made while stressed.
- [ ] **The maximum acceptable time to decide on rollback is agreed** — typically minutes, and stated before anyone is emotionally invested in the release.
- [ ] **A feature flag kill switch is tested if the release relies on one** — verify it takes effect without a redeploy.

## 4. Go/no-go, immediately before the window {#go-no-go-immediately-before-the-window}

- [ ] **The pipeline is still green and no new commits have landed on the release branch since the freeze.**
- [ ] **No active incident is in progress in this service or its critical dependencies** — check the incident tracker rather than assuming.
- [ ] **Dependent services and infrastructure are healthy** — a deploy into a degraded platform makes attribution impossible when something breaks.
- [ ] **No conflicting change is being applied at the same time** — infrastructure, DNS, certificate, or database maintenance from another team.
- [ ] **Current error rate, latency, and traffic levels are captured as the pre-deploy baseline** — you cannot judge a regression without the number from before.
- [ ] **All named roles are present and acknowledge the go decision explicitly** — silence is not consent.
- [ ] **The conductor states the abort criteria out loud before starting** — so everyone is holding the same decision rule.

{{< alert context="warning" text="**No-go conditions:** an active incident anywhere in the critical dependency chain, a rollback path that has not been verified, or a key person unavailable for the soak period. Any one of these is enough to postpone. Postponing costs a day; a bad release with nobody watching costs far more." />}}

## 5. Deploy window: execution {#deploy-window-execution}

- [ ] **Every step is announced in the release channel as it starts and as it completes** — with timestamps, because the timeline is what you will reconstruct later.
- [ ] **Backward-compatible schema migrations run and complete before the application deploy** — expand first, contract in a later release.
- [ ] **Migration duration and lock behaviour are watched live** — abort criteria for the migration are separate from those for the deploy.
- [ ] **The deploy uses the recorded artefact digest through the normal automated pipeline** — no manual steps, no local builds, no shortcuts because it is a special release.
- [ ] **The rollout is progressive and paused at the first stage for verification** — canary, one instance, or a small traffic percentage before proceeding.
- [ ] **Canary health is compared against the rest of the fleet, not against the baseline alone** — a fleet-wide change in traffic pattern otherwise looks like a canary regression.
- [ ] **Deploy progress is watched actively, not started and left** — the person who ran the command stays until the rollout completes.
- [ ] **If anything unexpected occurs, the rollout is paused rather than pushed through** — a paused rollout is cheap; a completed bad rollout is not.

## 6. Immediately after rollout: verification {#immediately-after-rollout-verification}

- [ ] **The running version is confirmed on every instance or replica** — a partially completed rollout serving two versions is a real and frequently missed state.
- [ ] **The critical user journey is exercised end to end by a human, not only by a synthetic probe** — log in, complete the primary action, confirm the result persisted.
- [ ] **Error rate, latency percentiles, and traffic are compared against the pre-deploy baseline** — looking at absolute values without the baseline hides small but real regressions.
- [ ] **Application logs are checked for new error signatures** — a new stack trace at low volume is often the first sign of a problem that grows with traffic.
- [ ] **Dependency health is checked from the caller's perspective** — connection pool saturation, downstream error rates, and queue depth.
- [ ] **Background jobs, scheduled tasks, and consumers are confirmed to be running and keeping up** — these fail quietly because no user is waiting on them.
- [ ] **Any feature flags intended to be enabled with this release are turned on one at a time, with verification between each.**
- [ ] **Cache and configuration invalidation has taken effect where required** — including content delivery network caches for front-end changes.

## 7. Soak period: monitoring {#soak-period-monitoring}

- [ ] **A defined soak duration is agreed before the release is declared done** — long enough to cover at least one full traffic cycle for the service, and never less than the time it takes for slow failure modes to appear.
- [ ] **A named person is actively watching dashboards for the soak, not relying on alerts alone** — alerts catch what you predicted; a human catches what you did not.
- [ ] **Resource saturation trends are watched** — memory growth, file descriptors, thread counts, and connection pools reveal leaks that error rates do not.
- [ ] **Latency at the tail is watched, not just the average** — p99 regressions affect the users most likely to complain publicly.
- [ ] **Business metrics are watched alongside technical ones** — sign-ups, orders, or messages sent, because a technically healthy release can still break the product.
- [ ] **Support and customer-facing channels are monitored for reports that dashboards will not show** — a broken payment flow for one card type is invisible in aggregate.
- [ ] **The on-call responder is briefed on what changed and what to do about it before the conductor stands down.**

## 8. Rollback decision {#rollback-decision}

- [ ] **The pre-agreed trigger conditions are applied as written, without renegotiation** — the whole purpose of setting them in advance is to remove judgement under pressure.
- [ ] **Rollback is the default response to an unexplained regression** — diagnose after service is restored, not before.
- [ ] **The decision is made within the agreed time limit** — an hour spent debugging a live regression is an hour of user impact you chose to accept.
- [ ] **If rollback is not possible because of data or schema changes, the forward-fix path is executed with the same discipline** — including its own verification and abort criteria.
- [ ] **Rollback execution is verified the same way the deploy was** — version confirmed everywhere, critical journey exercised, metrics compared to baseline.
- [ ] **The decision, its reasoning, and its timeline are recorded in the release channel as they happen.**
- [ ] **If rolling back, feature flags enabled during the release are also reverted** — a flag left on against an old binary is an untested combination.

## 9. Release closure {#release-closure}

- [ ] **The release is explicitly declared complete or rolled back in the release channel** — an ambiguous ending leaves the on-call responder unsure of the current state.
- [ ] **The deployed version, artefact digest, and timestamp are recorded in the change log or change management system.**
- [ ] **Stakeholders and support teams are notified of the outcome** — including anything that behaves differently than announced.
- [ ] **Temporary measures are reverted** — paused autoscaling, silenced alerts, extended timeouts, and any maintenance page.
- [ ] **Feature flags that are now permanent are scheduled for removal** — a flag that outlives its purpose is a permanently untested code path.
- [ ] **Anything that went wrong, including near misses, is recorded for the retrospective** — a release that worked out despite a scare still contains the lesson.
- [ ] **Improvements to this runbook are made while the details are fresh** — the best time to fix a runbook is the day you used it.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| T-1 day: scope and readiness | | | Pass / Pass with actions / Fail |
| T-1 day: communication and logistics | | | Pass / Pass with actions / Fail |
| T-1 day: rollback preparation | | | Pass / Pass with actions / Fail |
| Go/no-go decision | | | Pass / Pass with actions / Fail |
| Deploy window: execution | | | Pass / Pass with actions / Fail |
| Post-rollout verification | | | Pass / Pass with actions / Fail |
| Soak period monitoring | | | Pass / Pass with actions / Fail |
| Rollback decision | | | Pass / Pass with actions / Fail |
| Release closure | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with a named owner, and treat any unresolved item in the go/no-go or rollback preparation sections as a no-go rather than a follow-up.

## Related checklists

- [Production Readiness Review](/docs/devops/production-readiness/)
- [CI/CD Pipeline Review](/docs/devops/cicd-pipeline/)
- [Database Schema Migration](/docs/development/database-schema-migration/)
- [Incident Management](/docs/operations/incident-management/)
- [Change Management](/docs/itsm/change-management/)

## References

- [Google SRE Book — Release Engineering](https://sre.google/sre-book/release-engineering/)
- [Google SRE Workbook — Canarying Releases](https://sre.google/workbook/canarying-releases/)
- [Google SRE Book — Emergency Response](https://sre.google/sre-book/emergency-response/)
- [Martin Fowler — Feature Toggles](https://martinfowler.com/articles/feature-toggles.html)
- [Martin Fowler — Blue Green Deployment](https://martinfowler.com/bliki/BlueGreenDeployment.html)
