---
title: "New Service Bootstrap"
description: "Verify a newly created service has the repository, pipeline, and operational scaffolding it needs from day one."
icon: "add_box"
weight: 440
toc: true
tags: ["bootstrap", "scaffolding", "developer-experience", "backend"]
---

Everything a service is missing on its first day it will still be missing two years later, because nobody gets funded to retrofit a health endpoint. The point of a bootstrap checklist is to make the boring scaffolding a precondition of the first commit rather than a cleanup task. This covers the repository, the pipeline, and the operational hooks; the go-live decision itself belongs to the production readiness review.

{{< alert context="info" text="**Who runs this:** the engineer creating the service, with a platform or infrastructure reviewer. **When:** in the first week of the repository existing, well before any traffic. See also the [Production Readiness Review](/docs/devops/production-readiness/) for the launch gate." />}}

## 1. Justification and boundaries {#justification-and-boundaries}

- [ ] **There is a written reason this is a separate service rather than a module** — independent scaling, independent deployment cadence, or a genuine team boundary; "microservices" is not a reason.
- [ ] **The service owns its data and no other service reads its database directly** — a shared database turns two services into one distributed monolith with worse failure modes.
- [ ] **The bounded context is written in one paragraph** — what this service is responsible for, and explicitly what it is not.
- [ ] **The synchronous call graph is drawn** — each additional synchronous hop multiplies latency and failure probability, and the diagram is where that becomes visible.
- [ ] **The operational cost is estimated and someone has agreed to pay it** — a service is a permanent on-call, patching, and dependency-upgrade obligation, not just a repository.
- [ ] **The technology choice is on the team's supported list, or the exception is approved** — every new language adds a build toolchain, a security scanner, and a set of idioms the on-call must know at 3am.

## 2. Repository and code hygiene {#repository-and-code-hygiene}

- [ ] **The repository is created from the organisation's template where one exists** — templates carry the pipeline, the linting rules, and the security defaults that nobody remembers to add manually.
- [ ] **`README` gets a new engineer from clone to running locally in one documented sequence** — and someone who did not write it has followed it end to end.
- [ ] **`CODEOWNERS` names the owning team and branch protection requires review** — an unprotected default branch makes the review process advisory.
- [ ] **A licence and copyright header policy is set** — even for internal code, because internal code becomes external code more often than expected.
- [ ] **Formatter and linter run in CI and in a pre-commit hook** — formatting arguments in review are pure waste and are entirely automatable.
- [ ] **Dependencies are pinned with a committed lockfile** — floating versions make builds unreproducible and turn an upstream release into an unplanned incident.
- [ ] **Automated dependency update PRs are enabled** — batching a year of upgrades into one change is how services end up stranded on an unsupported runtime.
- [ ] **A `.gitignore` and a secret-scanning hook are in place from the first commit** — a credential committed once is committed forever in the history.

## 3. Configuration and secrets {#configuration-and-secrets}

- [ ] **All configuration comes from the environment, with no environment-specific code branches** — `if env == "production"` scattered through the codebase means staging never tests the production path.
- [ ] **The service fails fast at start-up when required configuration is missing or malformed** — validate on boot and exit non-zero rather than discovering it on the first request that needs it.
- [ ] **Secrets are fetched from a secret manager at runtime** — never baked into an image, never in the repository, and never in a plain environment variable committed to a manifest.
- [ ] **Credential rotation does not require a code change** — and the service either picks up rotated credentials or is documented as needing a restart.
- [ ] **Every configuration value is documented with its default, its type, and what it does** — undocumented tunables are the settings nobody dares change at 3am.
- [ ] **Local development uses non-production credentials against non-production data** — the first developer to point a local instance at production will not be the last.

## 4. Build and pipeline {#build-and-pipeline}

- [ ] **A single command builds, tests, and lints the service locally, and CI runs the identical command** — divergence between local and CI is the most common source of "works on my machine".
- [ ] **The build is reproducible from a clean checkout with no manual step** — no local tool installed by hand, no undocumented environment variable.
- [ ] **The container image is built from a pinned, minimal base and runs as a non-root user** — set this on the first image, because changing the user later breaks file permissions.
- [ ] **Artefacts are immutable and versioned by commit** — tagging only `latest` makes it impossible to say what is actually running.
- [ ] **Dependency, container, and static analysis scanning run in CI with a failing threshold** — a scanner whose findings never block anything is a report nobody reads.
- [ ] **CI runs in under ten minutes for the common path** — beyond that, engineers batch their pushes and the feedback loop stops working.
- [ ] **Deployment to at least one non-production environment is automated on merge** — a deploy path exercised many times a day is the one that still works during an incident.

## 5. Runtime scaffolding {#runtime-scaffolding}

- [ ] **Separate liveness and readiness endpoints exist** — liveness must not check downstream dependencies, or an outage in a dependency will make the orchestrator restart-loop healthy pods.
- [ ] **Graceful shutdown is implemented on the first day** — handle the termination signal, stop accepting new work, drain in-flight requests, and exit within the grace period.
- [ ] **Every outbound HTTP and database client has an explicit timeout** — library defaults are frequently infinite, and this is the single most common cause of cascading failure.
- [ ] **Connection pools are sized deliberately against the database's connection limit** — instance count multiplied by pool size is the number that matters, and it is usually discovered by exhausting it.
- [ ] **The service starts from cold with no manual step and no warm cache** — including after a full region restart.
- [ ] **A build version and commit hash are exposed at runtime** — through an endpoint or a start-up log line, so you can confirm what is deployed.
- [ ] **Resource requests and limits are set from a measured baseline, not copied from another service.**

{{< alert context="warning" text="**Blocking:** do not deploy a service whose outbound clients have no timeout, or whose liveness probe depends on a downstream service. Both turn a minor dependency wobble into a full outage, and both are trivial to fix now and painful to fix later." />}}

## 6. Observability from day one {#observability-from-day-one}

- [ ] **Logs are structured, at a configurable level, and written to standard output** — writing to files inside a container is a class of problem you never need to have.
- [ ] **A correlation or trace identifier is generated at the edge and propagated to every downstream call** — retrofitting propagation across an existing call graph is far harder than adding it now.
- [ ] **Request rate, error rate, and latency percentiles are exported from the first deployment** — you cannot detect a regression against a baseline you never collected.
- [ ] **Tracing is instrumented through the whole request path, not just the entry point** — a trace that stops at the first hop tells you nothing useful.
- [ ] **A dashboard exists and is linked from the `README`** — a dashboard nobody can find during an incident may as well not exist.
- [ ] **At least one alert on a user-visible symptom is configured before the first real traffic** — with a runbook link, even if the runbook is initially a stub.
- [ ] **No secret, token, or personal data is logged** — check specifically for code that logs whole request objects or authorisation headers.

## 7. Testing strategy {#testing-strategy}

- [ ] **The test pyramid is decided and enforced** — fast unit tests as the bulk, a thin layer of integration tests against real dependencies in containers, and a very small number of end-to-end tests.
- [ ] **Integration tests run against the real database engine and version** — an in-memory substitute will not reproduce locking, collation, or transaction semantics.
- [ ] **Contract tests exist for every API this service publishes or consumes** — so a breaking change fails your build rather than a downstream team's production.
- [ ] **The test suite is deterministic and runs in CI on every pull request** — a flaky suite is worse than no suite, because it teaches everyone to re-run rather than investigate.
- [ ] **Test data setup is isolated per test** — shared fixtures create order dependence, which surfaces as an unreproducible failure months later.
- [ ] **A smoke test runs against each environment after deployment** — verifying the deployed instance actually serves the critical path, not just that the pods are ready.

## 8. Ownership and operations {#ownership-and-operations}

- [ ] **The service is registered in the service catalogue with a named owning team** — a team, not an individual, and recorded somewhere machine-readable.
- [ ] **It is added to the on-call rotation's scope and the escalation path is recorded** — before launch, not after the first page goes to nobody.
- [ ] **A runbook stub exists covering start, stop, deploy, roll back, and how to tell whether it is actually healthy.**
- [ ] **Data classification is recorded** — whether the service stores personal data determines retention, encryption, and audit obligations, and it is much cheaper to decide now.
- [ ] **Backups and their restore procedure exist for any datastore the service owns** — and the restore has been performed at least once.
- [ ] **Cost allocation tags are applied to every resource the service creates** — service, team, and environment at minimum.
- [ ] **A decommissioning note records what would have to be deleted if this service is retired** — written while the answer is still known.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Justification and boundaries | | | Pass / Pass with actions / Fail |
| Repository and code hygiene | | | Pass / Pass with actions / Fail |
| Configuration and secrets | | | Pass / Pass with actions / Fail |
| Build and pipeline | | | Pass / Pass with actions / Fail |
| Runtime scaffolding | | | Pass / Pass with actions / Fail |
| Observability from day one | | | Pass / Pass with actions / Fail |
| Testing strategy | | | Pass / Pass with actions / Fail |
| Ownership and operations | | | Pass / Pass with actions / Fail |

Record each "Pass with actions" as a dated ticket; scaffolding deferred past the first release is rarely done at all.

## Related checklists

- [Production Readiness Review](/docs/devops/production-readiness/)
- [CI/CD Pipeline](/docs/devops/cicd-pipeline/)
- [Container Image](/docs/devops/container-image/)
- [Observability](/docs/operations/observability/)
- [Secrets Management](/docs/security/secrets-management/)

## References

- [The Twelve-Factor App](https://12factor.net/)
- [Google SRE Book — Service Best Practices](https://sre.google/sre-book/table-of-contents/)
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
