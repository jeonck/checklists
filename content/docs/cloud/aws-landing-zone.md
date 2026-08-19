---
title: "Cloud Landing Zone Setup"
description: "Verify a multi-account cloud foundation is secure, governable, and cost-attributable before workloads land on it."
icon: "account_balance"
weight: 310
toc: true
tags: ["landing-zone", "aws", "governance", "multi-account"]
---

A landing zone is the part of your cloud estate that nobody wants to rebuild later. Account boundaries, identity, network address space, and the audit trail are all decisions that become progressively more expensive to change once real workloads depend on them. This checklist is written primarily against AWS, with the Azure and Google Cloud equivalents called out where the concept maps cleanly, and assumes you are building the foundation before the first production workload rather than retrofitting one.

{{< alert context="info" text="**Who runs this:** the platform or cloud foundation team, with a reviewer from security and one from finance. **When:** before the first production workload is deployed, and re-run annually or after any change to the account structure." />}}

## 1. Account and organisation structure

- [ ] **A management account exists and runs no workloads** — the AWS Organizations management account (Azure: the root management group and its billing tenant; GCP: the Organization node) can bypass service control policies, so anything running in it is effectively unguarded.
- [ ] **Workloads are separated into accounts by blast radius, not by convenience** — one account per workload per environment is the defensible default; shared accounts make IAM the only boundary and IAM is the boundary most likely to be misconfigured.
- [ ] **Organisational units reflect policy, not the org chart** — group accounts by the guardrails they need (sandbox, workload, infrastructure, suspended) because OUs exist to attach policy, and reorganisations should not require moving accounts.
- [ ] **Dedicated accounts exist for log archive, security tooling, and shared network** — keeping the log archive in an account that application teams cannot reach is what makes the audit trail trustworthy during an incident.
- [ ] **A sandbox OU with a hard spend limit and detached from production networks exists** — without a sanctioned place to experiment, engineers will experiment in production accounts.
- [ ] **Account creation is automated and produces a fully baselined account** — AWS Control Tower Account Factory, Azure landing zone subscription vending, or GCP project factory; a manually created account will be missing exactly the control you needed.
- [ ] **Every account has a unique root email on a monitored distribution list, with MFA on root and no root access keys** — root account recovery goes to that mailbox, and a personal address means an offboarded employee owns your account.
- [ ] **Account closure and suspension procedures are documented** — including where the data goes and who confirms the account is empty before it is closed.

## 2. Identity and access

- [ ] **Human access is federated from a single identity provider** — AWS IAM Identity Center, Azure Entra ID, or Google Cloud Identity; long-lived IAM users with access keys are the single most common source of leaked cloud credentials.
- [ ] **No IAM users with static access keys exist outside a documented exception list** — and each exception has an owner, a rotation schedule, and a ticket for its removal.
- [ ] **Permission sets are defined centrally and assigned to groups, not individuals** — per-person policies drift, and nobody audits them.
- [ ] **Privileged access is time-bound and requires a justification** — AWS IAM Identity Center session duration plus approval workflow, Azure PIM, or GCP privileged access manager; standing administrator access is standing risk.
- [ ] **MFA is enforced at the identity provider for every human, including contractors and break-glass users.**
- [ ] **A break-glass path exists, is tested, and generates a high-priority alert when used** — credentials in a sealed vault, and a quarterly test that proves they still work.
- [ ] **Workloads use roles with a trust policy, never keys** — IAM roles for service accounts on EKS, Azure workload identity, or GCP Workload Identity Federation, so no credential is ever written to disk.
- [ ] **Cross-account trust policies constrain the external principal with a condition** — an unconditioned `sts:AssumeRole` trust on a third-party account ID is exploitable via the confused deputy problem unless an external ID is required.
- [ ] **Unused permissions are reviewed against access analyser findings** — IAM Access Analyzer or GCP Policy Intelligence recommendations will show you which of your broad policies nobody has ever exercised.

{{< alert context="danger" text="**Blocking:** if the management account root user does not have MFA, or if any account still has root access keys, stop and fix that before anything else on this list. Everything else assumes the top of the trust chain is sound." />}}

## 3. Network foundation

- [ ] **The IP address plan is allocated centrally and documented before the first VPC** — overlapping CIDR ranges cannot be peered or routed, and renumbering a live VPC means rebuilding it.
- [ ] **Address space is reserved for future regions, environments, and on-premises integration** — a /16 that seemed generous will not survive three environments plus Kubernetes pod CIDRs.
- [ ] **Connectivity uses a hub-and-spoke transit model** — AWS Transit Gateway, Azure Virtual WAN or hub VNet, GCP Network Connectivity Center; full-mesh VPC peering does not scale past a handful of accounts and gives you no central inspection point.
- [ ] **Every VPC has at least three availability zones for subnets carrying stateful services** — two-AZ designs lose quorum for anything using a three-node consensus protocol when one AZ fails.
- [ ] **Private subnets have no route to an internet gateway, and egress goes through NAT or a proxy** — and the NAT gateway cost is understood, because inter-AZ NAT traffic is a common six-figure surprise.
- [ ] **VPC endpoints exist for the high-volume services you actually use** — S3 and DynamoDB gateway endpoints are free and remove the largest source of NAT data processing charges.
- [ ] **Egress is filtered and logged** — an outbound-allow-all network turns a compromised container into a data exfiltration channel with no record of what left.
- [ ] **VPC flow logs are enabled to the central log archive with a retention period** — you cannot reconstruct lateral movement after the fact without them.
- [ ] **DNS resolution is centralised and split-horizon is deliberate** — Route 53 Resolver rules, Azure Private DNS zones, or Cloud DNS, so private endpoint names resolve consistently across accounts.

## 4. Preventive guardrails

- [ ] **Service control policies deny the actions no workload should ever perform** — disabling CloudTrail, deleting log buckets, or removing GuardDuty; Azure equivalent is Azure Policy deny effects at management group scope, GCP is organization policy constraints.
- [ ] **A region deny policy restricts the estate to approved regions** — this closes the most common cryptomining pattern, where compromised credentials spin up GPU instances in a region nobody monitors.
- [ ] **Guardrails are tested from inside a workload account, not asserted from the console** — attempt the denied action with a real role and confirm it fails.
- [ ] **Public access is blocked at the account level for object storage** — S3 Block Public Access at account scope, Azure storage account public network access disabled, GCP public access prevention enforced at the organization.
- [ ] **Default encryption and TLS-only access are enforced by policy, not by convention** — a deny on unencrypted `s3:PutObject` and on `aws:SecureTransport: false` catches the bucket that someone provisioned by hand.
- [ ] **IMDSv2 is required and instance metadata hop limit is set to 1** — the metadata service is how a server-side request forgery bug becomes credential theft.
- [ ] **Policies are attached at OU level and version-controlled** — an SCP edited in the console with no review is a production change with no change record.
- [ ] **A documented exception process exists with expiry dates** — guardrails without an escape hatch get disabled entirely the first time they block a launch.

## 5. Logging, detection, and audit trail

- [ ] **An organisation-wide trail captures management events in every account and region** — AWS CloudTrail organisation trail, Azure Activity Log diagnostic settings at management group, GCP Cloud Audit Logs at organization level; per-account trails will have gaps exactly where someone disabled one.
- [ ] **Log delivery goes to a bucket in the log archive account that workload accounts cannot write to or delete from** — with object lock or a retention policy so an attacker with account admin cannot erase their tracks.
- [ ] **Data events are enabled for the object storage and key stores that hold sensitive data** — management events do not tell you which objects were read, and that is the question after a breach.
- [ ] **Threat detection is enabled organisation-wide** — GuardDuty, Microsoft Defender for Cloud, or Security Command Center, with findings routed to a security account rather than left in each member account.
- [ ] **A configuration recorder tracks resource state and change history** — AWS Config, Azure Resource Graph change history, or GCP Asset Inventory feeds; drift detection is impossible without a record of what things looked like yesterday.
- [ ] **Findings route to a ticketing system or SIEM with an owner and an SLA** — a security dashboard nobody opens is a compliance artefact, not a control.
- [ ] **Log retention is set explicitly per log type and matches the legal and forensic requirement** — 90 days of flow logs is usually too short for a breach investigation and too long for the budget if you keep them in hot storage.

## 6. Encryption and key management

- [ ] **A key hierarchy is defined per environment and data classification** — one key for everything means one compromise for everything, and one key per bucket means an unmanageable key policy sprawl.
- [ ] **Customer-managed keys are used where key policy or rotation control matters** — provider-managed keys are fine for low-sensitivity data, but you cannot revoke access or prove separation of duties with them.
- [ ] **Key policies grant use, not administration, to workload roles** — a role that can schedule key deletion can destroy the data it protects.
- [ ] **Automatic key rotation is enabled and the rotation period is documented.**
- [ ] **Cross-account and cross-region key access is explicit and reviewed** — a snapshot copied to a DR region is unusable if the key does not exist there.
- [ ] **Secrets live in a secret manager with rotation, not in parameter stores as plaintext or in environment variables baked into images.**

## 7. Cost allocation and financial controls

- [ ] **A tagging standard exists with a small mandatory set** — owner, cost centre, environment, and service; a twenty-tag standard is a standard nobody follows.
- [ ] **Mandatory tags are enforced at provision time, not audited afterwards** — tag policies plus infrastructure-as-code defaults, because retroactive tagging of six months of resources never happens.
- [ ] **Cost allocation tags are activated in the billing console** — a tag that exists on the resource but is not activated for billing produces no cost report breakdown, and activation is not retroactive.
- [ ] **Consolidated billing is enabled with per-account budgets and alerts at multiple thresholds** — 50%, 80%, and 100% of forecast gives you time to react rather than a post-mortem.
- [ ] **An anomaly detector is enabled and routes to a human** — AWS Cost Anomaly Detection, Azure Cost Management alerts, or GCP budget alerts with a Pub/Sub trigger.
- [ ] **Someone owns the monthly bill review and it is on a calendar** — unowned cloud spend grows monotonically.

## 8. Automation, change control, and operations

- [ ] **The entire landing zone is defined as code in version control** — a console-built foundation cannot be reviewed, reproduced in a second region, or rebuilt after a mistake.
- [ ] **Infrastructure state is stored remotely with locking and versioning** — and the state backend itself lives in an account separate from the workloads it describes.
- [ ] **The pipeline that deploys the landing zone uses a role that workload teams cannot assume** — otherwise the guardrails and the thing being guarded share a trust boundary.
- [ ] **Drift detection runs on a schedule and reports differences to an owner** — manual console changes during an incident are legitimate; leaving them undocumented is not.
- [ ] **Service quotas are reviewed and raised ahead of need** — quota increases can take days, and discovering an elastic IP or VPC limit during a launch is avoidable.
- [ ] **A support plan exists at a tier that matches the recovery time objective** — basic support has no technical response commitment.
- [ ] **The landing zone has a documented owner, runbook, and upgrade path** — Control Tower and equivalent frameworks release new guardrails, and an unpatched foundation slowly diverges from the reference.

{{< alert context="warning" text="**Common mistake:** treating the landing zone as a one-off project. Assign a standing owner and a review cadence, or within a year you will have accounts created outside the factory, guardrails disabled for a launch that shipped six months ago, and no one who knows why." />}}

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Account and organisation structure | | | Pass / Pass with actions / Fail |
| Identity and access | | | Pass / Pass with actions / Fail |
| Network foundation | | | Pass / Pass with actions / Fail |
| Preventive guardrails | | | Pass / Pass with actions / Fail |
| Logging, detection, and audit trail | | | Pass / Pass with actions / Fail |
| Encryption and key management | | | Pass / Pass with actions / Fail |
| Cost allocation and financial controls | | | Pass / Pass with actions / Fail |
| Automation, change control, and operations | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with a named owner before the first production workload is onboarded.

## Related checklists

- [Cloud Security](/docs/security/cloud-security/)
- [Infrastructure as Code](/docs/devops/infrastructure-as-code/)
- [Secrets Management](/docs/security/secrets-management/)
- [Cloud Cost Optimisation](/docs/cloud/cloud-cost-optimization/)
- [Network Change](/docs/networking/network-change/)

## References

- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [AWS Organizations User Guide](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html)
- [Azure Cloud Adoption Framework — Landing Zones](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/)
- [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)
