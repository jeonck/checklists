---
title: "Infrastructure as Code Review"
description: "Verify a Terraform or equivalent infrastructure change is safe, reviewable, and reversible before it is applied."
icon: "terminal"
weight: 250
toc: true
tags: ["terraform", "iac", "cloud", "change-management"]
---

Infrastructure as code turns a cloud console mistake into a reviewable diff, but only if the review is real. The dangerous changes are rarely the ones that create resources; they are the ones that quietly destroy and recreate a database, widen a security group, or drift a state file away from reality. Use this when reviewing an infrastructure change and when assessing whether a repository's practices are sound.

{{< alert context="info" text="**Who runs this:** the engineer proposing the change plus a reviewer who did not write it. **When:** on every pull request that touches infrastructure code, with the module and state sections reviewed quarterly." />}}

## 1. Repository structure and module design {#repository-structure-and-module-design}

- [ ] **Environments are separated by directory or workspace with their own state, not by a single configuration with a conditional** — a shared configuration means a staging change can plan destructive edits to production.
- [ ] **Reusable modules are versioned and consumed by version constraint, not by a branch reference** — a module sourced from `main` changes under every consumer without a diff.
- [ ] **Module inputs are typed, validated, and documented** — an untyped map input is where undocumented behaviour accumulates.
- [ ] **Modules do not hard-code account IDs, regions, or environment names** — these belong in the calling configuration or the change is not reusable.
- [ ] **Provider and language versions are pinned with a lockfile committed to the repository** — a provider minor release can change default behaviour and produce an unexpected plan.
- [ ] **The repository has an owner and `CODEOWNERS` covers production configuration** — infrastructure code without an owner is applied by whoever is desperate enough.

## 2. State management {#state-management}

- [ ] **State is stored remotely with versioning enabled** — local state on a laptop is a single point of failure for the entire environment.
- [ ] **State locking is enabled and has been observed to work** — two concurrent applies against one state produce corruption that is painful to unwind.
- [ ] **The state backend is encrypted at rest and access is restricted to the automation identity and a small break-glass group** — state files contain secrets in plain text, including generated passwords and keys.
- [ ] **State backups exist and a restore from a previous state version has been rehearsed** — recovering from a bad `terraform state rm` depends on this.
- [ ] **State is split so that a blast radius is bounded** — one monolithic state for an entire organisation makes every change a high-risk change and every plan slow.
- [ ] **Manual state manipulation is documented and audited** — `state rm`, `import`, and `taint` are legitimate tools that also silently rewrite reality.

## 3. The plan output {#the-plan-output}

- [ ] **The plan has been read line by line, not just the summary counts** — the resource count tells you nothing about whether the right resources are changing.
- [ ] **Every destroy and every replace in the plan is justified in the pull request description** — a forced replacement of a database, a load balancer, or a key is the classic accidental outage.
- [ ] **The cause of each replacement is understood** — which attribute forces new, and whether a `lifecycle` block or a change of approach avoids it.
- [ ] **No unexplained changes appear that the diff does not account for** — unexplained drift means someone changed something by hand, and applying will revert it.
- [ ] **The plan is generated against the same state and variables that will be applied** — a plan file is saved and applied, rather than re-planning at apply time.
- [ ] **Data source lookups resolve to what you expect** — a filter that matches the wrong AMI or the wrong subnet produces a plan that looks harmless.

{{< alert context="danger" text="**Blocking:** never approve a plan containing a resource replacement you cannot explain. Replacement of stateful resources — databases, volumes, certificates, load balancers with static addresses — is data loss or an outage, and it is the most common cause of self-inflicted infrastructure incidents." />}}

## 4. Secrets and sensitive data {#secrets-and-sensitive-data}

- [ ] **No credential, key, or certificate is committed in a `.tf` or `.tfvars` file** — including in examples and test fixtures.
- [ ] **Secrets are read at apply time from a secret manager, not passed as plain variables** — and the reference, not the value, lives in the repository.
- [ ] **Outputs containing sensitive values are marked sensitive** — otherwise they are printed in logs and stored in plan artefacts.
- [ ] **Generated passwords are written directly to a secret manager rather than surfaced as outputs** — anything in state or output is readable by anyone with state access.
- [ ] **Plan artefacts are treated as sensitive and have a short retention** — a saved plan can contain the values being written.
- [ ] **Secret scanning runs on the repository history, not only on new commits** — infrastructure repositories accumulate old test credentials.

## 5. Security and compliance of the resources {#security-and-compliance-of-the-resources}

- [ ] **A policy-as-code check runs in the pipeline and blocks on violations** — encryption at rest, no public object storage, no unrestricted ingress, mandatory tags.
- [ ] **No security group, firewall rule, or bucket policy allows `0.0.0.0/0` except where explicitly documented and approved** — and never for administrative ports.
- [ ] **IAM policies are least-privilege with no wildcard action on a wildcard resource** — an over-broad role created by IaC is durable in a way an ad-hoc one is not.
- [ ] **Encryption at rest is enabled for every storage, database, and queue resource, with the key management model chosen deliberately.**
- [ ] **Logging and audit trails are enabled on the resources being created** — a resource created without logging is invisible during an incident.
- [ ] **Public exposure of any new endpoint is intentional and reviewed** — load balancers, managed database public accessibility, and function URLs each default differently.
- [ ] **Deletion protection is enabled on stateful production resources** — it is the last defence against a mistaken destroy.

## 6. Change safety and reversibility {#change-safety-and-reversibility}

- [ ] **The rollback path is described in the pull request** — reverting the commit is not always sufficient when data or DNS is involved.
- [ ] **`prevent_destroy` or the equivalent guard is set on irreplaceable resources** — databases, root DNS zones, and key management keys.
- [ ] **Changes are applied to a non-production environment first and the result observed** — an environment that never receives the change first is not a testing environment.
- [ ] **The change is small enough to reason about** — combining a provider upgrade, a refactor, and a functional change in one pull request makes the plan unreadable.
- [ ] **Resource moves use a `moved` block or a documented state move rather than a destroy and recreate** — renaming a resource in code otherwise destroys the real one.
- [ ] **Dependencies between changes are sequenced explicitly** — if a network change must land before a workload change, say so and order the merges.
- [ ] **Timing is considered** — infrastructure changes touching networking, DNS, or certificates should not land immediately before a peak period or a holiday.

## 7. Automation and access control {#automation-and-access-control}

- [ ] **Apply runs in a pipeline with a federated short-lived identity, not from a workstation with a long-lived key** — and the pipeline identity is the only one with production write access under normal conditions.
- [ ] **Production apply requires an explicit approval separate from the code review** — plan on pull request, apply on merge with a gate.
- [ ] **Direct human write access to production cloud accounts is break-glass only, time-limited, and alerted on** — otherwise drift is inevitable.
- [ ] **Drift detection runs on a schedule and reports differences between code and reality** — drift discovered during an incident is drift discovered too late.
- [ ] **Formatting, validation, and linting run automatically** — so reviews discuss behaviour rather than whitespace.
- [ ] **The pipeline fails closed** — an error in the plan step must not proceed to apply.

## 8. Documentation and operability {#documentation-and-operability}

- [ ] **The repository README explains how to run a plan, where state lives, and who to ask** — onboarding cost is a reliability property.
- [ ] **Every resource carries required tags** — owner, environment, service, and cost centre, enforced by policy rather than by convention.
- [ ] **Cost impact of the change is estimated for anything that adds significant resources** — instance families, provisioned throughput, and inter-zone traffic in particular.
- [ ] **Architecture documentation is updated in the same change** — a diagram that lags the code is worse than no diagram.
- [ ] **Deprecated resources and modules are removed rather than left commented out** — commented-out infrastructure is ambiguous during an incident.
- [ ] **The change is recorded in the change management system where one applies** — with the plan output attached as evidence.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Repository structure and module design | | | Pass / Pass with actions / Fail |
| State management | | | Pass / Pass with actions / Fail |
| The plan output | | | Pass / Pass with actions / Fail |
| Secrets and sensitive data | | | Pass / Pass with actions / Fail |
| Security and compliance of the resources | | | Pass / Pass with actions / Fail |
| Change safety and reversibility | | | Pass / Pass with actions / Fail |
| Automation and access control | | | Pass / Pass with actions / Fail |
| Documentation and operability | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with a named owner before the change is applied to production.

## Related checklists

- [CI/CD Pipeline Review](/docs/devops/cicd-pipeline/)
- [AWS Landing Zone](/docs/cloud/aws-landing-zone/)
- [Cloud Security](/docs/security/cloud-security/)
- [Change Management](/docs/itsm/change-management/)
- [Network Change](/docs/networking/network-change/)

## References

- [HashiCorp — Terraform documentation](https://developer.hashicorp.com/terraform/docs)
- [HashiCorp — Terraform style guide](https://developer.hashicorp.com/terraform/language/style)
- [Open Policy Agent documentation](https://www.openpolicyagent.org/docs/latest/)
- [AWS Well-Architected Framework — Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)
