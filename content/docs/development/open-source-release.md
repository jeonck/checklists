---
title: "Open Source Release"
description: "Verify a project is legally, technically, and operationally ready to be published as open source."
icon: "public"
weight: 450
toc: true
tags: ["open-source", "licensing", "release", "governance"]
---

Publishing a repository is a one-way door. Git history is permanent, package registries generally do not allow a version to be quietly replaced, and the first thing a stranger does with new code is read it for credentials. This checklist covers the legal review, the history scrub, the packaging, and the governance commitments you are making by putting your name on a public project.

{{< alert context="info" text="**Who runs this:** the maintaining team, with sign-off from legal or open source programme office and a security reviewer. **When:** at least two weeks before the intended publication date, because licence and history findings are slow to resolve." />}}

## 1. Legal and licensing {#legal-and-licensing}

- [ ] **The licence is chosen deliberately and approved by whoever owns that decision** — permissive and copyleft licences make materially different commitments, and changing later requires the agreement of every contributor.
- [ ] **A `LICENSE` file with the full, unmodified licence text is at the repository root** — package managers, scanners, and GitHub's licence detection all key off it.
- [ ] **Every dependency's licence is compatible with your chosen licence** — a copyleft dependency inside a permissively licensed project is the finding that most often stops a release.
- [ ] **Vendored, copied, or generated third-party code retains its original licence and attribution** — including code pasted from an answer site or produced from a licensed template.
- [ ] **A `NOTICE` file exists where any dependency licence requires attribution** — Apache-2.0 in particular obliges you to carry it forward.
- [ ] **Contributions from outside the organisation are covered by a CLA or a DCO sign-off** — decide before the first pull request, not after.
- [ ] **Trademarks, product names, and logos are used according to policy** — the code licence does not grant trademark rights, and the distinction confuses almost everyone.
- [ ] **Patent implications have been reviewed if the project implements anything patented in-house.**

## 2. History and secret scrubbing {#history-and-secret-scrubbing}

- [ ] **The entire git history has been scanned for secrets, not just the current tree** — a key deleted in a later commit is still trivially retrievable from the history the moment the repository is public.
- [ ] **Automated secret scanning has been run with more than one tool** — detectors have different rule sets, and the cost of a second run is minutes.
- [ ] **Any credential that ever appeared in the history is rotated regardless of scrub outcome** — assume it is compromised; a rewritten history does not help if a fork or a mirror already exists.
- [ ] **Internal hostnames, IP ranges, internal URLs, and infrastructure identifiers are removed** — these are reconnaissance material and are rarely caught by secret scanners.
- [ ] **Customer names, personal data, and real datasets are removed from fixtures, tests, and documentation** — test fixtures are the most overlooked source of a personal data disclosure.
- [ ] **Internal commit messages and branch names have been reviewed** — they routinely reference ticket systems, incidents, security findings, and individuals by name.
- [ ] **If the history cannot be cleaned, publish from a squashed initial commit** — losing history is preferable to publishing a leak, and it is the honest default for code extracted from a monorepo.
- [ ] **Push protection and secret scanning are enabled on the public repository before the first push** — the protection has to exist before the commit that would need it.

{{< alert context="danger" text="**Blocking:** never publish before rotating every credential that has appeared anywhere in the repository history. Public repositories are indexed by automated scrapers within minutes, and a rewritten history does not un-publish anything that was already fetched or forked." />}}

## 3. Documentation {#documentation}

- [ ] **The `README` states what the project does and who it is for in the first three lines** — a visitor decides whether to keep reading in about ten seconds.
- [ ] **Installation and a minimal working example are copy-pasteable and have been tested on a clean machine** — by someone who did not write them.
- [ ] **Supported versions, platforms, and runtimes are stated explicitly** — otherwise every unsupported combination becomes an issue in your tracker.
- [ ] **The project's scope and non-goals are documented** — the cheapest way to decline a feature request is to have said no to that category in advance.
- [ ] **`CONTRIBUTING.md` covers how to build, test, and submit a change, and what makes a change acceptable** — including whether you want unsolicited feature pull requests at all.
- [ ] **A `CODE_OF_CONDUCT.md` is present with a working, monitored reporting address** — a reporting alias nobody reads is worse than none.
- [ ] **Documentation is versioned with the code** — documentation describing an unreleased API is a support burden from day one.

## 4. Code and repository quality {#code-and-repository-quality}

- [ ] **The code builds from a clean clone with only the documented prerequisites** — no internal package registry, no internal base image, no company-only tooling.
- [ ] **All internal dependencies have been removed or replaced with public equivalents** — an internal utility library referenced in `go.mod` or `package.json` will break every external build immediately.
- [ ] **Comments, identifiers, and error messages are in the project's stated language and free of internal jargon** — internal codenames make code unreadable to outsiders.
- [ ] **Tests run in CI on the public repository, on the platforms you claim to support** — a badge that reflects an internal pipeline nobody can see is not evidence.
- [ ] **CI on pull requests from forks is configured safely** — workflows that run untrusted code with access to repository secrets are a well-understood supply chain attack.
- [ ] **The default branch is protected and direct pushes are disabled, including for maintainers.**
- [ ] **Dependency update automation and vulnerability alerts are enabled** — public projects are scanned by everyone, and a stale vulnerable dependency will be reported publicly.

## 5. Versioning and releases {#versioning-and-releases}

- [ ] **The project follows semantic versioning and says so** — and the meaning of the major, minor, and patch positions for this project is documented.
- [ ] **The initial version signals stability honestly** — publishing `1.0.0` commits you to a stable public interface; use `0.x` if you are not ready for that.
- [ ] **What constitutes the public API is defined** — which packages, modules, and symbols are covered by the compatibility promise, and which are internal.
- [ ] **A `CHANGELOG.md` is maintained in a keep-a-changelog style with the changes grouped by type** — generated commit logs are not a changelog because they are written for the wrong audience.
- [ ] **Releases are tagged, signed, and immutable** — and the tag matches the artefact published to the package registry.
- [ ] **The release process is automated and documented** — a release only one person can perform is a bus-factor problem and an inconsistency generator.
- [ ] **Build provenance or an SBOM is published with the artefacts** — consumers increasingly require it, and generating it later for old releases is impossible.
- [ ] **The package registry namespace and account are owned by the organisation, not by an individual's personal account.**

## 6. Security posture {#security-posture}

- [ ] **A `SECURITY.md` states how to report a vulnerability and what response time to expect** — without it, reporters disclose publicly by default.
- [ ] **A private disclosure channel exists and is monitored by more than one person** — a single monitored inbox means a report sits unread while the reporter's disclosure clock runs.
- [ ] **An embargo and coordinated disclosure process is agreed before you need it** — the first report is a bad time to invent a process.
- [ ] **Static analysis and dependency scanning run on the public repository** — outsiders will scan it anyway, and it is better to find the issue before they file it publicly.
- [ ] **Publishing credentials for the package registry are scoped, stored in a secret manager, and protected by multi-factor authentication** — a compromised publish token lets an attacker ship malware under your name.
- [ ] **Maintainer accounts with publish rights all have phishing-resistant multi-factor authentication enabled.**
- [ ] **The threat model of running this code is documented if it processes untrusted input** — users need to know what guarantees you do and do not make.

## 7. Governance and sustainability {#governance-and-sustainability}

- [ ] **Maintainers are named and there is more than one with full access** — a single-maintainer project with a single publish key is one lost laptop from being abandoned.
- [ ] **The decision-making process for accepting changes is written down** — even if it is "the maintainers decide", stating it prevents a lot of friction.
- [ ] **Expected response times for issues and pull requests are stated** — under-promising is fine; silence is what drives contributors away.
- [ ] **The support commitment is explicit** — which versions receive fixes, and for how long.
- [ ] **The internal cost of maintenance has an owner and allocated time** — an open source project without funded maintenance time degrades into an unanswered issue tracker.
- [ ] **An archival or transfer plan exists for when the project is no longer maintained** — deciding in advance is far kinder to users than silent abandonment.

## 8. Launch and after {#launch-and-after}

- [ ] **A pre-publication dry run has been done on a private repository with the same settings** — including a full release to a registry's test channel where one exists.
- [ ] **Issue templates and a pull request template are configured** — they measurably improve the quality of what arrives.
- [ ] **Repository topics, description, and homepage are set** — this is how the project is found.
- [ ] **A rollback plan exists for the release itself** — know your registry's rules on unpublishing and yanking before you need them, because most do not allow a version to be replaced.
- [ ] **Someone is on hand for the first week to answer issues** — the first days after publication determine whether the project attracts contributors or a backlog.
- [ ] **Internal teams that depend on the code know it is now public** — so they stop making internal-only assumptions in their pull requests.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Legal and licensing | | | Pass / Pass with actions / Fail |
| History and secret scrubbing | | | Pass / Pass with actions / Fail |
| Documentation | | | Pass / Pass with actions / Fail |
| Code and repository quality | | | Pass / Pass with actions / Fail |
| Versioning and releases | | | Pass / Pass with actions / Fail |
| Security posture | | | Pass / Pass with actions / Fail |
| Governance and sustainability | | | Pass / Pass with actions / Fail |
| Launch and after | | | Pass / Pass with actions / Fail |

Legal and secret-scrubbing sections must be a clear pass; everything else may carry a dated follow-up ticket with a named owner.

## Related checklists

- [Secrets Management](/docs/security/secrets-management/)
- [Security Code Review](/docs/security/security-code-review/)
- [CI/CD Pipeline](/docs/devops/cicd-pipeline/)
- [Code Review](/docs/development/code-review/)
- [Release Day](/docs/devops/release-day/)

## References

- [Semantic Versioning](https://semver.org/)
- [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
- [Choose an Open Source License](https://choosealicense.com/)
- [Open Source Guides — Starting an Open Source Project](https://opensource.guide/starting-a-project/)
- [Contributor Covenant](https://www.contributor-covenant.org/)
