---
title: "Cloud Security Posture"
description: "Verify the identity, network, data, and logging controls of a cloud account before and after workloads land in it."
icon: "admin_panel_settings"
weight: 120
toc: true
tags: ["cloud", "iam", "posture", "hardening"]
---

Cloud breaches rarely involve breaking the provider. They involve an over-permissive role, a storage bucket that was made public for a demo, or a long-lived access key committed to a repository. This checklist reviews the posture of a cloud account or subscription across identity, network, data, logging, and workload configuration. It is written to apply to AWS, Azure, and Google Cloud; substitute the local name for each concept as you go.

{{< alert context="info" text="**Who runs this:** the cloud platform team, reviewed by security. **When:** when a new account or subscription is created, quarterly thereafter, and whenever a new workload type is introduced." />}}

## 1. Account structure and guardrails {#account-structure-and-guardrails}

- [ ] **Workloads are separated into accounts or subscriptions by environment and blast radius** — an account is the strongest isolation boundary the provider offers, and it is cheap.
- [ ] **Organisation-level policies deny the actions no team should ever take** — disabling logging, leaving the approved regions, or deleting audit trails.
- [ ] **The management or root account holds no workloads** — it exists to govern, and its compromise is unrecoverable.
- [ ] **The root or global administrator credential has hardware MFA, no access keys, and a documented break-glass procedure** — with use of it alerting the security team immediately.
- [ ] **Unused regions are disabled or explicitly denied** — attackers mine cryptocurrency in the region nobody watches.
- [ ] **Every resource carries owner, environment, and data-classification tags, enforced at creation** — untagged resources cannot be triaged during an incident.

## 2. Identity and access management {#identity-and-access-management}

- [ ] **Humans authenticate through a central identity provider with SSO and MFA** — not through per-account local users.
- [ ] **Long-lived access keys are eliminated in favour of short-lived, federated credentials** — workload identity federation and instance roles remove the secret that gets leaked.
- [ ] **Any remaining static keys are inventoried, rotated on a schedule, and alerted on when unused for 90 days** — a forgotten key is an unmonitored back door.
- [ ] **Roles are scoped by both action and resource** — a policy allowing all actions on all resources is not a policy.
- [ ] **Wildcard administrative permissions are limited to a named, small, reviewed set of principals** — and their use is logged and alerted.
- [ ] **Privilege-escalation paths have been analysed, not just permission counts** — the ability to pass a role, modify a policy, or update a function's code is administrative access in disguise.
- [ ] **Permissions are right-sized from observed usage, and access reviews happen at least quarterly** — entitlements accumulate; nothing removes them automatically.
- [ ] **Cross-account trust policies name specific principals and use an external ID where a third party is involved** — a trust policy open to a whole account is open to everyone in it.

{{< alert context="danger" text="**Blocking:** a role that can create or attach IAM policies is effectively an administrator, however narrow its other permissions look. Treat every such role as privileged and review it by name." />}}

## 3. Network security {#network-security}

- [ ] **Workloads run in private subnets with egress through a controlled path** — public IPs on compute should be the rare, justified exception.
- [ ] **Security groups and firewall rules avoid ingress from 0.0.0.0/0 except on deliberate public listeners** — pay special attention to 22, 3389, and database ports.
- [ ] **Administrative access uses a bastion, session manager, or zero-trust proxy rather than open SSH or RDP.**
- [ ] **Private connectivity to managed services uses private endpoints** — keeping storage and database traffic off the public internet and off the NAT bill.
- [ ] **Flow logs are enabled and retained** — you cannot answer what an attacker reached without them.
- [ ] **Public-facing applications sit behind a WAF and DDoS protection with tuned, non-default rules.**
- [ ] **The internet-facing attack surface is enumerated and matched against what should be exposed** — run an external scan rather than trusting the diagram.

## 4. Data protection {#data-protection}

- [ ] **No storage bucket, blob container, or snapshot is publicly readable unless it is deliberately a public asset store** — with account-level public access blocking switched on as the default.
- [ ] **Encryption at rest is enabled everywhere, with customer-managed keys where the data classification requires it.**
- [ ] **Key policies restrict who can use and who can administer each key, and key deletion has a waiting period** — separating use from administration prevents a single compromised role destroying data.
- [ ] **Encryption in transit is enforced by policy, not just available** — deny unencrypted access at the bucket and database level.
- [ ] **Backups exist in a separate account with immutability or object lock** — ransomware that reaches your account will delete backups it can reach.
- [ ] **Data residency and cross-region replication match the legal commitments made to customers.**
- [ ] **Sensitive data stores are discovered and classified with tooling, not by memory** — shadow copies in analytics buckets are the usual surprise.

## 5. Logging, detection, and response {#logging-detection-and-response}

- [ ] **The provider's control-plane audit log is enabled in all regions and delivered to a separate, restricted account.**
- [ ] **Log storage is write-once for the retention period** — an attacker's first act is to stop or delete the logs.
- [ ] **The provider's threat detection service is enabled and its findings route to a monitored queue** — findings nobody triages are worse than none, because they create false comfort.
- [ ] **Alerts exist for the high-signal events** — root login, IAM policy change, security group opened to the internet, CloudTrail or activity log disabled, and mass data download.
- [ ] **Posture management scans run continuously against a benchmark** — the CIS foundations benchmark is a reasonable baseline.
- [ ] **Logs are retained long enough for a realistic investigation** — the median breach is discovered months after entry.

## 6. Compute and workload hardening {#compute-and-workload-hardening}

- [ ] **Machine images are built from a hardened, patched base and rebuilt regularly** — patching a long-lived instance in place drifts from the source of truth.
- [ ] **Instance metadata service v2, or the equivalent hardening, is enforced** — session-oriented metadata access is the main defence against SSRF stealing role credentials.
- [ ] **Containers run as non-root with a read-only filesystem and no unnecessary capabilities.**
- [ ] **Serverless functions each have their own minimal execution role** — a shared role gives every function the union of all permissions.
- [ ] **Vulnerability scanning covers images and running workloads, with a defined remediation SLA by severity.**
- [ ] **Managed database services are not publicly accessible and have deletion protection enabled.**

## 7. Infrastructure as code and change control {#infrastructure-as-code-and-change-control}

- [ ] **All infrastructure is defined as code and applied through a pipeline, not through the console** — console changes are invisible to review and reappear as drift.
- [ ] **Console write access in production is broken-glass only, and its use raises an alert.**
- [ ] **The pipeline runs policy-as-code checks that fail the build on a violation** — a scanner that only warns will be ignored under deadline.
- [ ] **State files are stored encrypted with restricted access and locking** — Terraform state contains secrets and a map of everything you own.
- [ ] **The pipeline's own cloud identity is scoped and its plan and apply stages are separated** — the deployment role is usually the most powerful identity in the account.
- [ ] **Drift is detected and reported on a schedule.**

## 8. Cost, quota, and resilience signals {#cost-quota-and-resilience-signals}

- [ ] **Budget alerts exist per account** — a sudden spend spike is often the first observable sign of a compromise.
- [ ] **Service quotas are known and monitored** — hitting a quota during an incident removes your ability to scale out of it.
- [ ] **The account has a tested recovery plan for the loss of a region and for the loss of the account itself.**
- [ ] **Third-party integrations with cloud access are inventoried, scoped, and reviewed** — vendor roles are a standard supply-chain entry point.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Account structure and guardrails | | | Pass / Pass with actions / Fail |
| Identity and access management | | | Pass / Pass with actions / Fail |
| Network security | | | Pass / Pass with actions / Fail |
| Data protection | | | Pass / Pass with actions / Fail |
| Logging, detection, and response | | | Pass / Pass with actions / Fail |
| Compute and workload hardening | | | Pass / Pass with actions / Fail |
| Infrastructure as code and change control | | | Pass / Pass with actions / Fail |
| Cost, quota, and resilience signals | | | Pass / Pass with actions / Fail |

Record the account or subscription identifier alongside the sign-off, because posture is per-account and does not transfer.

## Related checklists

- [AWS Landing Zone](/docs/cloud/aws-landing-zone/)
- [Infrastructure as Code](/docs/devops/infrastructure-as-code/)
- [Secrets Management](/docs/security/secrets-management/)
- [Security Incident Response](/docs/security/incident-response/)
- [Backup and Recovery](/docs/operations/backup-and-recovery/)

## References

- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)
- [AWS Well-Architected Framework — Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
- [Microsoft Cloud Adoption Framework](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/)
- [Google Cloud Architecture Framework — Security](https://cloud.google.com/architecture/framework/security)
- [NIST SP 800-53 Security and Privacy Controls](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
