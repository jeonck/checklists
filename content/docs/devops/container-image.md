---
title: "Container Image Hardening"
description: "Verify a container image is minimal, reproducible, and safe to run before it is promoted to production."
icon: "inventory_2"
weight: 230
toc: true
tags: ["containers", "docker", "supply-chain", "hardening"]
---

Most container images in production are far larger, far more privileged, and far older than anyone intended. Each extra package is attack surface you will be asked about after an incident, and each unpinned base tag is a change you did not review. Work through this before an image is promoted beyond a development environment, and again whenever the base image changes.

{{< alert context="info" text="**Who runs this:** the team that owns the Dockerfile, with a reviewer from platform or security for images that run in production. **When:** at image creation, at any base image change, and on a recurring schedule for long-lived images." />}}

## 1. Base image selection {#base-image-selection}

- [ ] **The base image comes from a known, maintained publisher** — an official image, a vendor-supported distribution image, or your own internal base, rather than an arbitrary registry account.
- [ ] **The base image is pinned by digest, not by tag** — `FROM node:22-slim` moves under you; `FROM node:22-slim@sha256:...` is a decision you can review.
- [ ] **The smallest viable base is used** — distroless, Alpine, or a slim variant, since packages you never installed cannot be vulnerable.
- [ ] **The base image's support lifecycle outlasts the planned life of the service** — building on a distribution release that reaches end of life in three months guarantees unpatchable CVEs.
- [ ] **A rebuild cadence is defined for picking up base image patches** — an image built once and never rebuilt accumulates vulnerabilities even if your own code never changes.
- [ ] **The architecture set is explicit** — multi-architecture images are built deliberately, so an arm64 node does not silently pull an emulated amd64 image.

## 2. Build hygiene {#build-hygiene}

- [ ] **A multi-stage build separates compilation from runtime** — compilers, package managers, and build caches must not ship to production.
- [ ] **The final stage copies only the artefacts it needs** — copying the whole build workspace defeats the point of multi-stage.
- [ ] **A `.dockerignore` excludes `.git`, local environment files, credentials, and test fixtures** — build context leakage is a routine cause of secrets inside images.
- [ ] **Package installation and cache cleanup happen in the same layer** — a cleanup in a later layer removes files from the filesystem view but not from the image size or history.
- [ ] **Layer ordering puts rarely changing steps first** — dependency installation before source copy, so a code change does not invalidate the whole cache.
- [ ] **No package manager remains usable at runtime where it can be removed** — an image where an attacker can `apt-get install` their tooling is a much friendlier environment for them.
- [ ] **The image builds without `--privileged` or a mounted Docker socket** — build steps requiring the host daemon are a privilege escalation path from CI into the host.

## 3. Secrets and build-time data {#secrets-and-build-time-data}

- [ ] **No secret is passed as a build argument** — `ARG` values are recorded in image history and readable by anyone who can pull the image.
- [ ] **Private registry or repository credentials use build secret mounts** — BuildKit secret mounts are not persisted into any layer.
- [ ] **The image history has been inspected for accidental secret inclusion** — a file deleted in a later layer is still present in the earlier one and still extractable.
- [ ] **No `.env`, `.npmrc`, `.netrc`, or cloud credential file is present in the final image** — verify by listing the filesystem, not by reading the Dockerfile.
- [ ] **Runtime configuration comes from the environment or a mounted secret, not baked into the image** — an image containing production endpoints and keys cannot be reused or safely shared.
- [ ] **Private base images are pulled with credentials scoped to the pipeline** — not a shared personal registry token.

## 4. Runtime user and filesystem {#runtime-user-and-filesystem}

- [ ] **The image declares a non-root `USER` with a fixed numeric UID** — a username alone does not satisfy orchestrator policies that check `runAsNonRoot`.
- [ ] **The UID is outside the host's reserved range and does not collide with a privileged host user** — a common convention is a UID above 10000.
- [ ] **The application does not need to write to the root filesystem** — everything writable is an explicit volume or `tmpfs`, so the container can run with a read-only root.
- [ ] **No file in the image is setuid or setgid unless justified** — strip these bits during the build and record any exception.
- [ ] **File ownership and permissions are set at build time, not by a start-up script running as root** — an entrypoint that starts privileged and drops later is weaker than never being privileged.
- [ ] **The image contains no shell where the workload does not require one** — a distroless runtime removes the most convenient post-exploitation tooling.

## 5. Image content and dependencies {#image-content-and-dependencies}

- [ ] **The package list has been reviewed and unnecessary packages removed** — debugging tools, editors, and network utilities are for a debug image, not the production one.
- [ ] **Application dependencies are installed from a lockfile with integrity hashes** — so the image content is a function of the commit, not of the registry at build time.
- [ ] **A software bill of materials is generated at build and stored alongside the image** — this is how you answer "are we affected?" during the next ecosystem-wide CVE.
- [ ] **The image is scanned for operating system and language-level vulnerabilities before promotion** — many scanners default to OS packages only and miss the application layer entirely.
- [ ] **Critical and high findings are fixed or have a recorded, dated exception** — with the compensating control written down, not implied.
- [ ] **Licence obligations of bundled dependencies have been checked** — copyleft components in a distributed image create obligations that are expensive to discover late.

{{< alert context="warning" text="**Common mistake:** scanning the image once at build time and never again. A CVE published tomorrow applies to the image running today, so scan images in the registry on a schedule and alert on newly discovered findings in what is deployed, not only in what is being built." />}}

## 6. Image metadata and identity {#image-metadata-and-identity}

- [ ] **The image carries OCI standard labels** — source repository, revision, build timestamp, and version, so a running container can be traced to a commit.
- [ ] **Tags are immutable in the registry** — overwriting a tag makes rollback and forensics unreliable.
- [ ] **Deployments reference images by digest** — a tag tells you the intent, a digest tells you the bytes.
- [ ] **A meaningful versioning scheme is used, and `latest` is not deployed anywhere** — `latest` is the fastest way to lose track of what is running.
- [ ] **The image is signed and the signature is verifiable** — with the verification wired into admission control rather than performed by hand.
- [ ] **Build provenance attestations are attached to the image** — recording which builder, which source commit, and which parameters produced it.

## 7. Runtime configuration in the image {#runtime-configuration-in-the-image}

- [ ] **`ENTRYPOINT` uses exec form so the application is PID 1 and receives signals** — a shell form entrypoint swallows `SIGTERM` and turns every deploy into a hard kill after the grace period.
- [ ] **Zombie process reaping is handled** — either the application reaps children correctly or a minimal init process is used.
- [ ] **`EXPOSE` documents the ports actually served, and nothing listens on an undocumented port** — surprises here become surprises in network policy.
- [ ] **Application logs go to stdout and stderr** — writing to files inside the container hides logs from the platform's collector.
- [ ] **A health check endpoint exists in the application and is documented** — the orchestrator's probe configuration depends on it.
- [ ] **The default working directory and configuration paths are documented** — operators should not have to read the Dockerfile to mount a config file.

## 8. Registry and distribution {#registry-and-distribution}

- [ ] **Push access to production repositories is limited to the pipeline identity** — no individual should be able to push an image that admission control will accept.
- [ ] **The registry requires authentication for pulls of private images and does not permit anonymous listing** — public registry repositories routinely leak internal service names and configuration.
- [ ] **Retention policies remove untagged and superseded images** — while retaining every digest that is still referenced by a running workload or a rollback plan.
- [ ] **Image promotion between environments copies the same digest** — never a rebuild, or staging validated something production never ran.
- [ ] **Registry availability is considered in the failure model** — a node that cannot pull cannot recover, so pull policies and caching are chosen deliberately.
- [ ] **Access to the registry is audited** — pushes, deletes, and permission changes are logged and reviewed.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Base image selection | | | Pass / Pass with actions / Fail |
| Build hygiene | | | Pass / Pass with actions / Fail |
| Secrets and build-time data | | | Pass / Pass with actions / Fail |
| Runtime user and filesystem | | | Pass / Pass with actions / Fail |
| Image content and dependencies | | | Pass / Pass with actions / Fail |
| Image metadata and identity | | | Pass / Pass with actions / Fail |
| Runtime configuration in the image | | | Pass / Pass with actions / Fail |
| Registry and distribution | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with a named owner before the image is promoted to production.

## Related checklists

- [CI/CD Pipeline Review](/docs/devops/cicd-pipeline/)
- [Kubernetes Deployment](/docs/devops/kubernetes-deployment/)
- [Secrets Management](/docs/security/secrets-management/)
- [Cloud Security](/docs/security/cloud-security/)

## References

- [Docker — Best practices for building images](https://docs.docker.com/build/building/best-practices/)
- [OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
- [NIST SP 800-190 — Application Container Security Guide](https://csrc.nist.gov/pubs/sp/800/190/final)
- [OCI Image Format Specification](https://github.com/opencontainers/image-spec)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)
