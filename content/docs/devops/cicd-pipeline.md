---
title: "CI/CD Pipeline Review"
description: "Verify a build and deployment pipeline is reproducible, secure, and trustworthy before teams depend on it."
icon: "account_tree"
weight: 220
toc: true
tags: ["ci-cd", "supply-chain", "automation", "devops"]
---

A pipeline is production infrastructure. It holds credentials to every environment you own, it decides what code becomes a running artefact, and when it is wrong it is wrong for every service at once. Review it the way you would review a service that has write access to production, because that is exactly what it is.

{{< alert context="info" text="**Who runs this:** the team that owns the pipeline, with a reviewer from platform or security. **When:** when a new pipeline is created, after any change to its credentials or permissions, and at least annually for pipelines that deploy to production." />}}

## 1. Source control and change flow

- [ ] **The default branch is protected against direct pushes** — including for administrators, or the protection is decorative.
- [ ] **Merging requires a passing pipeline and at least one approving review** — and approvals are dismissed when new commits are pushed, otherwise a reviewer approves one diff and a different one merges.
- [ ] **`CODEOWNERS` covers the pipeline definition itself** — a change to the deploy workflow deserves more scrutiny than a change to a README.
- [ ] **Force-push and branch deletion are disabled on protected branches** — recovering the true history after a force-push to main is far harder than preventing it.
- [ ] **Commits are traceable to a person** — signed commits or a verified identity, so a compromised token cannot quietly author a change attributed to someone else.
- [ ] **Every deployable commit is reachable from a tag or release record** — you must be able to answer "what exactly is running?" without guessing.

## 2. Build reproducibility

- [ ] **A clean checkout on a fresh runner produces the same artefact** — any dependence on a warm cache, a pre-installed tool, or a developer's machine is a latent build break.
- [ ] **All dependencies are pinned by version and, where the ecosystem supports it, by digest or hash** — a lockfile that resolves floating ranges at build time is not pinning.
- [ ] **The lockfile is committed and the build fails if it is out of date** — `npm ci`, `pip install --require-hashes`, `go mod verify` and equivalents rather than a permissive install.
- [ ] **Base images and build tool versions are pinned rather than tracking `latest`** — an upstream tag moving under you turns a green pipeline red for reasons unrelated to your change.
- [ ] **The build has no network access to unpinned sources at package time** — pulling a script from a URL during the build makes the artefact a function of the internet's mood.
- [ ] **Build outputs are written to a fresh workspace each run** — leftover files from a previous build are a classic source of artefacts that cannot be recreated.

## 3. Pipeline credentials and permissions

- [ ] **The pipeline authenticates to cloud providers with short-lived OIDC federation, not a long-lived access key stored as a repository secret** — a leaked static key is valid until someone notices; a federated token expires in minutes.
- [ ] **Each job requests the minimum token scope it needs** — a test job does not need write permission to packages or deployments.
- [ ] **Secrets are scoped per environment** — the staging deploy job must not be able to read production credentials.
- [ ] **Deployment credentials are held by a protected environment with required reviewers, not by the workflow file** — otherwise any contributor who can edit a workflow can reach production.
- [ ] **Workflows triggered by forked pull requests cannot read secrets** — the `pull_request_target` trigger and equivalents are the single most exploited CI misconfiguration.
- [ ] **Secrets are masked in logs and the masking has been tested** — try echoing a secret through base64 in a scratch branch and confirm it is redacted.
- [ ] **Credential rotation is documented and has been performed at least once** — an unrotatable credential is a permanent liability.

{{< alert context="danger" text="**Blocking:** a pipeline that exposes production credentials to workflows runnable from a fork is remotely exploitable by any user of the platform. Fix this before anything else on the list." />}}

## 4. Third-party actions and runners

- [ ] **Every third-party action or plugin is pinned to a full commit SHA, not a tag** — tags are mutable, so a pinned tag is an invitation to a supply-chain compromise.
- [ ] **The set of allowed actions is restricted by policy** — an allowlist of verified publishers and your own organisation, rather than the whole marketplace.
- [ ] **Self-hosted runners for public repositories are ephemeral and isolated** — a persistent runner shared with untrusted pull requests leaks the previous job's state.
- [ ] **Runner images are patched and rebuilt on a schedule** — a runner is a long-lived machine with credentials, and it ages like any other host.
- [ ] **Caches are keyed so that untrusted branches cannot poison a cache used by trusted builds** — cache entries are executable inputs.
- [ ] **The pipeline pulls container images by digest, not by mutable tag** — `:latest` in a deployment step means you cannot say what you shipped.

## 5. Testing and quality gates

- [ ] **Unit, integration, and contract tests run on every pull request and block merge on failure** — a test suite that only runs after merge is a report, not a gate.
- [ ] **Flaky tests are quarantined with an owner and a deadline, not retried silently** — automatic retries hide real race conditions until they reach production.
- [ ] **Database migrations are exercised against a realistic schema in CI** — a migration that has only ever run on an empty table has not been tested.
- [ ] **Test data contains no production personal data** — synthetic or masked fixtures only, since CI logs and artefacts are widely readable.
- [ ] **Total pipeline duration for a pull request is measured and kept short enough that people do not batch changes** — slow pipelines cause large, risky merges.
- [ ] **A failing pipeline cannot be bypassed by a manual override without an audit record** — emergency overrides need to exist and need to leave a trace.

## 6. Security scanning in the pipeline

- [ ] **Dependency scanning runs on every build and fails on new critical vulnerabilities** — with an explicit, time-limited exception process rather than a permanently ignored report.
- [ ] **Static analysis runs on changed code and its findings block merge at an agreed severity** — a scanner whose output nobody reads is theatre.
- [ ] **Secret scanning runs on the diff and on the full history** — push protection stops the next leak, history scanning finds the one from last year.
- [ ] **Container images are scanned after build and before promotion** — scanning only the Dockerfile misses everything the base image contributes.
- [ ] **Infrastructure-as-code templates are scanned for misconfiguration** — public buckets and open security groups are cheaper to catch here than in a cloud audit.
- [ ] **Scanner findings have an owner and an SLA by severity** — an unbounded backlog of findings is indistinguishable from no scanning.

## 7. Artefact management and provenance

- [ ] **Artefacts are built once and promoted through environments unchanged** — rebuilding per environment means staging never tested what production runs.
- [ ] **Every artefact is immutable in the registry** — tag overwrites must be disabled, or a rollback can silently deliver different bytes.
- [ ] **A software bill of materials is generated and stored with each artefact** — when the next widely used library has a critical CVE, you need to answer "are we affected?" in minutes.
- [ ] **Build provenance is recorded and verifiable** — an attestation binding the artefact digest to the source commit and the builder, following SLSA levels.
- [ ] **The deployment step verifies the artefact signature or attestation before rolling out** — provenance you never check is metadata, not a control.
- [ ] **Registry retention and cleanup policies exist** — untagged layers accumulate cost, and stale images get deployed by accident.

## 8. Deployment and rollback

- [ ] **Deployment to production is a distinct, separately authorised step** — not a side effect of merging.
- [ ] **The pipeline can deploy a specific previous artefact version on demand** — rollback must not require rebuilding from an old commit.
- [ ] **Concurrency control prevents two deployments to the same environment at once** — overlapping rollouts produce a state nobody designed.
- [ ] **Rollouts are progressive with automated abort on error-rate or latency regression** — canary or percentage-based, with the abort criteria defined before the deploy.
- [ ] **A break-glass deployment path exists and is documented** — including how to deploy when the CI platform itself is down.
- [ ] **Every deployment writes an audit record** — who, what artefact digest, which environment, and when.

## 9. Pipeline reliability and maintenance

- [ ] **Pipeline failures alert the owning team, not just the commit author** — a broken shared pipeline blocks everyone.
- [ ] **Pipeline definitions are version controlled and reviewed like application code** — no click-configured jobs that exist only in the CI platform's database.
- [ ] **Success rate and duration are tracked over time** — a slowly degrading pipeline is a slowly degrading release cadence.
- [ ] **The pipeline is restorable if the CI platform account is lost** — configuration in the repository, secrets in a secret manager with a documented restore.
- [ ] **Unused workflows, runners, and credentials are removed on a schedule** — dormant automation with live credentials is a favourite foothold.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Source control and change flow | | | Pass / Pass with actions / Fail |
| Build reproducibility | | | Pass / Pass with actions / Fail |
| Pipeline credentials and permissions | | | Pass / Pass with actions / Fail |
| Third-party actions and runners | | | Pass / Pass with actions / Fail |
| Testing and quality gates | | | Pass / Pass with actions / Fail |
| Security scanning in the pipeline | | | Pass / Pass with actions / Fail |
| Artefact management and provenance | | | Pass / Pass with actions / Fail |
| Deployment and rollback | | | Pass / Pass with actions / Fail |
| Pipeline reliability and maintenance | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with a named owner before the pipeline is approved for production deployments.

## Related checklists

- [Production Readiness Review](/docs/devops/production-readiness/)
- [Container Image Hardening](/docs/devops/container-image/)
- [Secrets Management](/docs/security/secrets-management/)
- [Security Code Review](/docs/security/security-code-review/)
- [Release Day Runbook](/docs/devops/release-day/)

## References

- [NIST Secure Software Development Framework (SSDF)](https://csrc.nist.gov/projects/ssdf)
- [SLSA — Supply-chain Levels for Software Artifacts](https://slsa.dev/)
- [OWASP CI/CD Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/CI_CD_Security_Cheat_Sheet.html)
- [GitHub — Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions)
- [Sigstore documentation](https://docs.sigstore.dev/)
