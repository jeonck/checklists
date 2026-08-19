---
title: "Backup and Recovery"
description: "Verify backups exist, are protected from your own mistakes, and have been restored successfully at least once."
icon: "backup"
weight: 550
toc: true
tags: ["backup", "disaster-recovery", "resilience", "data"]
---

Nobody wants backups; everybody wants restores. The distinction matters because a backup job that reports success every night proves only that a job ran, not that the data inside it is readable, complete, or recoverable within the window the business assumes. This checklist works from what you are protecting, through how it is stored and secured, to the only test that counts.

{{< alert context="info" text="**Who runs this:** the service owner together with whoever operates the data platform, reviewed by someone outside the team. **When:** before go-live, after any change to storage or retention, and at least once a year regardless." />}}

## 1. Scope and inventory

- [ ] **Every data store the service depends on is listed** — databases, object storage, search indexes, message queues with durable state, and the caches that cannot be rebuilt from source.
- [ ] **Configuration and infrastructure state are in scope, not just user data** — Terraform state, secret manager contents, DNS zones, IAM policies, and CI/CD configuration are all restore-critical.
- [ ] **Each dataset has a named owner and a classification** — the owner decides retention and approves restores, and the classification determines encryption and access requirements.
- [ ] **Data that is deliberately not backed up is documented as an accepted loss** — derived data and rebuildable caches are legitimate exclusions, but only when written down.
- [ ] **Dependencies between datasets are recorded** — restoring a database without the object storage its rows reference produces a consistent-looking but broken system.
- [ ] **The inventory is generated or verified automatically** — a hand-maintained list will silently miss the database that was created last quarter.

## 2. Recovery objectives

- [ ] **An RPO and RTO are agreed per dataset with the business, in writing** — engineering cannot invent an acceptable data loss window on the business's behalf.
- [ ] **The objectives are derived from actual impact, not from what the current tooling happens to deliver** — a 24-hour RPO for payments data is a decision, and it should be a conscious one.
- [ ] **The measured recovery time is compared against the RTO after every restore test** — most teams discover their real RTO is several times their stated one.
- [ ] **Recovery time includes everything, not just the restore command** — detection, decision, provisioning, restore, replay, validation, and cutover all consume the window.
- [ ] **Objectives for a full regional loss are separate from those for a single-table deletion** — they are different scenarios with different mechanisms and different costs.
- [ ] **The cost of meeting each objective is known and accepted** — a one-minute RPO is achievable and expensive, and the trade-off belongs to the data owner.

## 3. Backup implementation

- [ ] **Backup frequency actually satisfies the stated RPO** — a nightly snapshot cannot deliver a four-hour RPO, no matter what the policy document says.
- [ ] **Point-in-time recovery is enabled where the RPO is shorter than the snapshot interval** — continuous transaction log archiving is what closes the gap between snapshots.
- [ ] **Backups are consistent, not merely copied** — use the database's own snapshot mechanism or quiesce writes, since a file-level copy of a running database is often unrecoverable.
- [ ] **Related datasets are captured at a coordinated point in time** — otherwise restoring produces referential inconsistency that is discovered days later.
- [ ] **Backup jobs alert on failure and on unexpected success characteristics** — a job that completes in a tenth of the usual time and produces a tenth of the usual bytes has failed silently.
- [ ] **Missing backups are alerted on, not just failing ones** — a job that stopped being scheduled generates no failure event at all.
- [ ] **Backup duration and size are trended** — steady growth predicts the day the job stops fitting inside its window.

## 4. Storage, isolation, and retention

- [ ] **At least one copy is in a different region or physical location** — a backup stored beside the primary shares its failure domain.
- [ ] **At least one copy is in an account or subscription with separate credentials** — an attacker or a runaway script with production credentials must not be able to delete the backups.
- [ ] **Object lock or immutability is enabled for the retention period** — this is the control that survives ransomware and a compromised administrator, and it cannot be added after the fact.
- [ ] **Deletion of backups requires a second approval and is logged** — and lifecycle rules that expire data are reviewed, since a misconfigured rule deletes quietly and permanently.
- [ ] **Retention satisfies both recovery needs and legal obligations** — long enough to recover from corruption discovered late, and short enough to satisfy data protection commitments.
- [ ] **Retention is enforced by an automated job rather than by intention** — unbounded retention is a compliance liability and an unbounded bill.
- [ ] **Storage costs are attributed and reviewed** — backup storage grows monotonically and is a common source of budget surprises.

{{< alert context="danger" text="**Blocking:** if the credentials that run production can also delete the backups, you do not have backups. Immutable storage in a separately credentialed account is the minimum bar for anything you would be unwilling to lose." />}}

## 5. Encryption and access control

- [ ] **Backups are encrypted at rest and in transit** — including snapshots, exports, and any temporary staging bucket used during the copy.
- [ ] **Encryption keys are stored separately from the backups and are themselves backed up** — an unrecoverable key makes an intact backup worthless, which is the most avoidable form of total data loss.
- [ ] **Key rotation does not orphan older backups** — verify that a backup taken before the last rotation is still decryptable, by decrypting one.
- [ ] **Access to backups is least-privilege and audited** — a full backup is the most concentrated copy of your data that exists, and it is frequently the least protected.
- [ ] **Restores into non-production environments anonymise or mask personal data** — copying production data into a development account is a common and serious breach path.
- [ ] **Break-glass access to backups is documented and tested** — including how to restore when the normal identity provider is the thing that is down.

## 6. Restore testing

- [ ] **A full restore has been performed end to end, into a clean environment** — an untested backup is a hypothesis, and the failure rate of first restores is high.
- [ ] **Restore tests run on a schedule, at least quarterly, and after every major version upgrade** — a database engine upgrade can silently break compatibility with older backup formats.
- [ ] **The restore is validated by data content, not by job exit code** — check row counts, run a checksum or reconciliation query, and exercise a real application read path.
- [ ] **Partial restores are tested as well as full ones** — the far more common real-world case is one table or one bucket prefix deleted by mistake.
- [ ] **Point-in-time recovery is tested by restoring to a specific timestamp** — including replaying transaction logs, which is the step that usually turns out to be misconfigured.
- [ ] **The restore is performed by someone who did not build it, following only the runbook** — this is how you find the undocumented step that lives in one person's head.
- [ ] **Measured restore time is recorded and compared to the RTO** — with any gap raised as a ticket rather than quietly normalised.

## 7. Runbooks and decision-making

- [ ] **A restore runbook exists per dataset with exact commands** — during a data-loss incident nobody should be reading vendor documentation for the first time.
- [ ] **The runbook states who authorises a restore** — overwriting production with a backup is itself destructive and needs a named decision-maker.
- [ ] **Rolling back to a backup and rolling forward are both covered** — including how to reconcile writes that happened after the restore point.
- [ ] **The runbook is stored where it can be read while the primary systems are down** — an incident guide that only exists in the wiki hosted on the failed cluster is not a guide.
- [ ] **Vendor support paths and account identifiers are recorded in the runbook** — some restores require the provider, and finding the support contract number under pressure wastes the window.
- [ ] **Communication expectations during a restore are defined** — restores are slow, and stakeholders need a cadence rather than silence.

## 8. Failover and disaster scenarios

- [ ] **Loss of an entire region has a documented and rehearsed response** — including how DNS, certificates, and secrets are made available in the recovery region.
- [ ] **The recovery environment can actually be provisioned** — check service quotas, instance availability, and image replication in the target region before you need them.
- [ ] **Failback is planned, not just failover** — returning to the primary region with reconciled data is usually the harder half and is frequently unplanned.
- [ ] **Ransomware and malicious deletion are treated as distinct scenarios** — they require immutable copies and a known-clean recovery point, not simply the most recent backup.
- [ ] **Dependencies on third parties are included in the disaster plan** — an identity provider, payment processor, or DNS provider outage needs a documented response of its own.
- [ ] **A full disaster recovery exercise is run at least annually with the outcome recorded** — with findings ticketed exactly as they would be after a real incident.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Scope and inventory | | | Pass / Pass with actions / Fail |
| Recovery objectives | | | Pass / Pass with actions / Fail |
| Backup implementation | | | Pass / Pass with actions / Fail |
| Storage, isolation, and retention | | | Pass / Pass with actions / Fail |
| Encryption and access control | | | Pass / Pass with actions / Fail |
| Restore testing | | | Pass / Pass with actions / Fail |
| Runbooks and decision-making | | | Pass / Pass with actions / Fail |
| Failover and disaster scenarios | | | Pass / Pass with actions / Fail |

Record the date and measured duration of the most recent successful restore alongside the sign-off, and treat any area without a completed restore test as Fail.

## Related checklists

- [Disaster Recovery Drill](/docs/itsm/disaster-recovery-drill/)
- [Production Readiness Review](/docs/devops/production-readiness/)
- [Database Schema Migration](/docs/development/database-schema-migration/)
- [Secrets Management](/docs/security/secrets-management/)
- [Capacity Planning](/docs/operations/capacity-planning/)

## References

- [NIST SP 800-34 Rev. 1 — Contingency Planning Guide for Federal Information Systems](https://csrc.nist.gov/pubs/sp/800/34/r1/upd1/final)
- [AWS Well-Architected Framework — Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)
- [Google Cloud Architecture Framework — Disaster Recovery Planning Guide](https://cloud.google.com/architecture/dr-scenarios-planning-guide)
- [Azure Well-Architected Framework — Reliability](https://learn.microsoft.com/en-us/azure/well-architected/reliability/)
- [Google SRE Book — Data Integrity: What You Read Is What You Wrote](https://sre.google/sre-book/data-integrity/)
