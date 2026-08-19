---
title: "Secrets Management"
description: "Verify credentials are generated, stored, delivered, rotated, and revoked without ever landing in a repository or an image."
icon: "key"
weight: 140
toc: true
tags: ["secrets", "credentials", "rotation", "appsec"]
---

Secrets leak the same way every time: someone needed a credential in a hurry, put it somewhere convenient, and nobody ever moved it. This checklist covers the whole lifecycle of a secret — how it is created, where it lives, how it reaches the workload that needs it, how it is rotated, and how it is destroyed. Work through it per system rather than per organisation, because one exemplary service and one hardcoded database password in the next repository still adds up to a breach.

{{< alert context="info" text="**Who runs this:** the owning engineering team, with a platform or security engineer for the storage and rotation sections. **When:** at service bootstrap, before any new integration credential is issued, and after any suspected exposure." />}}

## 1. Inventory and classification {#inventory-and-classification}

- [ ] **Every secret the system uses is listed with its type, owner, consumer, and blast radius** — you cannot rotate what nobody has written down.
- [ ] **Each secret has a stated impact if disclosed** — read-only analytics credentials and production database root are not managed the same way.
- [ ] **Shared secrets are distinguished from per-service secrets** — a credential used by six services cannot be rotated without coordinating six deployments.
- [ ] **Secrets held by third parties and vendors are included in the inventory** — the webhook signing key you gave a partner is still your secret.
- [ ] **Human-held secrets are separated from machine-held secrets** — they need different storage, different rotation, and different revocation triggers.

## 2. Eliminating secrets where possible {#eliminating-secrets-where-possible}

- [ ] **Workload identity or federated credentials replace static keys wherever the platform supports it** — the safest secret is the one that never exists.
- [ ] **Cloud resources are accessed through attached roles rather than access keys** — instance profiles, managed identities, and workload identity federation all remove the long-lived key.
- [ ] **Service-to-service authentication uses short-lived tokens or mutual TLS** — a certificate with a lifetime of hours is a much smaller prize than a static API key.
- [ ] **Database access uses IAM authentication or short-lived generated credentials where available.**
- [ ] **Any remaining static credential has a written justification and a review date** — this forces the exception to be revisited instead of becoming permanent.

## 3. Storage {#storage}

- [ ] **All secrets live in a dedicated secret manager or vault** — not in environment files in a repository, not in a wiki page, not in a pinned chat message.
- [ ] **Access to each secret is authorised per principal and per secret path** — a single policy granting read on everything defeats the purpose of the vault.
- [ ] **Every read of a production secret is audit-logged with the principal, time, and source.**
- [ ] **The vault's own unseal keys, root token, or master credential are split, stored offline, and never used for routine operations.**
- [ ] **Kubernetes Secrets are encrypted at rest with an external key provider and are not the source of truth** — base64 is encoding, not encryption, and any principal that can read the namespace can read them.
- [ ] **Secrets are never written to a container image, a build artefact, or an AMI** — image layers persist even when the file is deleted in a later layer.
- [ ] **Configuration management and IaC state files do not contain plaintext secrets** — Terraform state is a common and overlooked plaintext store.

## 4. Delivery to workloads {#delivery-to-workloads}

- [ ] **Secrets are fetched at runtime, or injected by the platform, rather than baked in at build time** — build-time injection ties every rotation to a rebuild.
- [ ] **The workload's identity for retrieving secrets is not itself a stored secret** — otherwise you have only moved the problem one level down.
- [ ] **Secrets in environment variables are avoided where the platform allows a file or API-based alternative** — environment variables leak through crash dumps, process listings, and child processes.
- [ ] **Secrets are held in memory for as long as needed and not written to disk, temp files, or swap-backed caches.**
- [ ] **The application fails closed when the secret store is unavailable** — falling back to a cached or default credential is how expired secrets stay in use.
- [ ] **Developers use per-developer credentials against non-production systems** — nobody needs a production secret on a laptop for local development.

## 5. Rotation and expiry {#rotation-and-expiry}

- [ ] **Every secret has a defined maximum lifetime and a rotation owner** — an unrotated credential accumulates every past employee who ever saw it.
- [ ] **Rotation is automated and has been executed successfully at least once** — a documented manual procedure that nobody has run does not work.
- [ ] **The system supports two valid credentials at once during rotation** — without overlap, rotation is an outage, so it will be postponed indefinitely.
- [ ] **Expiry is monitored and alerts fire well before the deadline** — certificates and signing keys expire on a schedule that is entirely predictable and still causes outages.
- [ ] **Rotation is triggered by events as well as by schedule** — team member departure, vendor incident, or any suspected exposure.
- [ ] **Rotation runbooks state the rollback path** — including what breaks if the new credential is wrong.

## 6. Preventing leakage {#preventing-leakage}

- [ ] **Secret scanning runs pre-commit, in CI, and across full repository history** — a secret removed in the latest commit is still in the history and must be treated as compromised.
- [ ] **Push protection blocks a commit containing a detected secret rather than reporting it afterwards.**
- [ ] **Application logs, error trackers, and traces are checked for credential leakage** — headers, connection strings, and full request bodies are the usual carriers.
- [ ] **CI/CD logs mask secret values and pull requests from forks cannot access repository secrets** — an untrusted contributor should never be able to echo your deployment key.
- [ ] **Secrets are never pasted into chat, tickets, or documents, and there is a supported alternative channel for the times someone must share one.**
- [ ] **Public repositories, package artefacts, and client bundles are scanned for embedded keys** — an API key shipped in a mobile app or JavaScript bundle is public by definition.

{{< alert context="danger" text="**Treat any secret that has ever been committed to version control as compromised, even in a private repository, even if the commit was amended.** Rotate it. Removing the commit hides the evidence, not the exposure." />}}

## 7. Revocation and incident handling {#revocation-and-incident-handling}

- [ ] **A documented procedure exists to revoke any single secret within minutes** — and the person on call knows where it is.
- [ ] **The consumer list per secret is accurate enough to predict what revocation will break** — this is the inventory from section 1 earning its keep.
- [ ] **Break-glass credentials exist, are stored offline, and their use raises an immediate alert.**
- [ ] **Post-exposure procedure includes reviewing the audit log for use of the secret, not only rotating it** — rotation stops future abuse and tells you nothing about past abuse.
- [ ] **Departing staff trigger revocation of every credential they could have read, not only their own accounts** — shared secrets they had access to must be rotated.

## 8. Governance and verification {#governance-and-verification}

- [ ] **Access to production secrets is reviewed at least quarterly against current role membership.**
- [ ] **A periodic exercise verifies that a randomly chosen secret can actually be rotated end to end** — treat it like a restore test.
- [ ] **Metrics track secrets past their maximum age, secrets with no owner, and secrets with excessive readers.**
- [ ] **New services inherit the secret management pattern by default through a template or platform module** — voluntary adoption produces exactly the gaps this checklist is meant to close.
- [ ] **Developers have documented, low-friction guidance for the common cases** — people work around a process that makes doing the right thing slow.
- [ ] **The vault or secret manager itself is covered by backup, availability, and disaster recovery planning** — it becomes a hard dependency for every service that fetches at runtime.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Inventory and classification | | | Pass / Pass with actions / Fail |
| Eliminating secrets where possible | | | Pass / Pass with actions / Fail |
| Storage | | | Pass / Pass with actions / Fail |
| Delivery to workloads | | | Pass / Pass with actions / Fail |
| Rotation and expiry | | | Pass / Pass with actions / Fail |
| Preventing leakage | | | Pass / Pass with actions / Fail |
| Revocation and incident handling | | | Pass / Pass with actions / Fail |
| Governance and verification | | | Pass / Pass with actions / Fail |

Attach the current secret inventory to the sign-off, since every other row depends on it being complete.

## Related checklists

- [Security Code Review](/docs/security/security-code-review/)
- [Cloud Security Posture](/docs/security/cloud-security/)
- [CI/CD Pipeline](/docs/devops/cicd-pipeline/)
- [Employee IT Offboarding](/docs/itsm/employee-it-offboarding/)
- [TLS Certificate](/docs/networking/tls-certificate/)

## References

- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_CheatSheet.html)
- [NIST SP 800-57 Recommendation for Key Management](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final)
- [HashiCorp Vault Documentation](https://developer.hashicorp.com/vault/docs)
- [AWS Secrets Manager User Guide](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)
- [Kubernetes — Good Practices for Kubernetes Secrets](https://kubernetes.io/docs/concepts/security/secrets-good-practices/)
