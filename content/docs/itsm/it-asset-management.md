---
title: "IT Asset Management"
description: "Verify hardware, software, and licences are inventoried, owned, patched, reconciled, and disposed of safely."
icon: "inventory"
weight: 950
toc: true
tags: ["itsm", "asset-management", "compliance", "lifecycle"]
---

Asset management is unglamorous until the moment it matters: a vulnerability advisory lands and you need to know which machines are affected, a vendor opens a licence audit, or a leaver fails to return a laptop that nobody recorded issuing. The register is not the goal — an accurate register that reconciles against the systems that actually see the assets is. Work through this as a periodic review of the asset management practice rather than as a one-off inventory exercise.

{{< alert context="info" text="**Who runs this:** the asset manager with IT operations, finance, and the security owner. **When:** quarterly for the full review, with reconciliation running monthly and on every joiner, mover, and leaver event." />}}

## 1. Hardware inventory

- [ ] **Every asset has a unique identifier that is physically attached and recorded** — asset tag and serial number, because model and user name are not unique and change over time.
- [ ] **The register covers all categories, not just laptops** — servers, network equipment, mobile devices, security keys, monitors, and the equipment sitting in home offices.
- [ ] **Assets in storage, in repair, and in transit have distinct states in the register** — an asset marked assigned but sitting in a cupboard is invisible loss.
- [ ] **The register is reconciled against automated discovery from the management platform** — a device in the endpoint console that is absent from the register is unmanaged inventory, and the reverse is a device that has stopped reporting.
- [ ] **A physical spot check of a sample is performed periodically and the error rate is recorded** — the error rate tells you whether the register is trustworthy; without it, you are guessing.
- [ ] **Purchase date, warranty end, cost, and cost centre are recorded** — these drive refresh planning, warranty claims, and depreciation, and they are painful to reconstruct later.
- [ ] **Cloud and virtual assets are inventoried alongside physical ones** — instances, volumes, and managed services are assets with owners and costs even though nobody can trip over them.

## 2. Software inventory and discovery

- [ ] **Installed software is discovered automatically on every managed endpoint and server** — a manual software inventory is out of date the day after it is compiled.
- [ ] **The inventory records version and edition, not just product name** — licence entitlements and vulnerability advisories are both version-specific.
- [ ] **SaaS subscriptions are inventoried alongside installed software** — most organisations now spend more on subscriptions than on installed licences and track them far worse.
- [ ] **Unapproved and unknown software is flagged and triaged rather than merely logged** — a discovery report nobody acts on is an audit trail of negligence.
- [ ] **Shadow IT is actively hunted through expense data, single sign-on logs, and network telemetry** — the tools bought on a personal card are the ones holding company data with no contract.
- [ ] **Open-source components in internally built software are inventoried in a software bill of materials** — when the next widely exploited library vulnerability lands, this is the only thing that answers are we affected quickly.
- [ ] **Browser extensions and unmanaged plugins are inventoried on endpoints** — an extension with read access to every page is an unreviewed data processor.

## 3. Ownership and lifecycle

- [ ] **Every asset has a named custodian and a named business owner** — the custodian holds it, the owner decides about it, and both fields being empty is the norm in neglected registers.
- [ ] **Lifecycle states are defined and every asset is in exactly one** — ordered, in stock, assigned, in repair, retired, disposed.
- [ ] **State transitions are triggered by events, not by periodic tidying** — assignment on issue, unassignment on return, retirement on decommission, each with a date.
- [ ] **Refresh cycles are defined per asset class and planned against budget** — an unplanned fleet refresh is both a capital shock and a support burden.
- [ ] **Assets assigned to leavers are cleared within a defined period** — an asset still assigned to someone who left three months ago means either the asset is lost or the register is wrong, and both need action.
- [ ] **Loan and temporary assignments have an expected return date and are chased** — loaner equipment without a return date becomes permanently issued equipment.

## 4. Licence compliance and true-up risk

- [ ] **Entitlements are recorded from contracts and matched against measured deployment** — the licence position is entitlement minus consumption, and you need both numbers to know where you stand.
- [ ] **Licence metrics are understood per product: per user, per device, per core, or per concurrent session** — core-based and virtualisation-based metrics are where unexpected liabilities are created.
- [ ] **Virtualisation and cloud hosting terms are checked against the vendor's rules** — running certain licensed software on shared cloud infrastructure can multiply the required licence count.
- [ ] **Over-deployment is quantified and remediated before a vendor audit finds it** — self-identified over-deployment is bought at list price; audit-identified over-deployment comes with penalties and back-maintenance.
- [ ] **Under-utilised licences and dormant SaaS seats are identified and reclaimed at renewal** — paid seats for departed staff are the most common avoidable IT spend.
- [ ] **Renewal dates and notice periods are tracked with reminders well ahead of auto-renewal** — a missed 90-day notice period locks in another year at the same seat count.
- [ ] **Open-source licence obligations for distributed software are reviewed** — copyleft obligations in a shipped product are a legal exposure, not a procurement one.
- [ ] **Proof of entitlement documents are stored centrally and retrievably** — an auditor will not accept we definitely bought it.

{{< alert context="warning" text="**Audit exposure:** the expensive part of a licence audit is rarely the licences you knew about. It is virtualised or cloud-hosted deployments counted under a different metric than you assumed, and non-production environments that the vendor counts as production. Check those two things before a vendor does." />}}

## 5. End-of-life and patching

- [ ] **End-of-support dates for operating systems, firmware, databases, and appliances are recorded in the register** — so the estate can be queried for what stops receiving fixes in the next 12 months.
- [ ] **Assets past end of support are listed, risk-assessed, and either replaced or formally accepted** — an unsupported system that nobody has decided about is an unowned risk.
- [ ] **Patch compliance is measured per asset class against a defined target and time window** — critical patches within days, routine patches within weeks, with the actual figures reported.
- [ ] **Assets that have not reported to the management platform recently are investigated** — a device silent for 30 days is either lost, retired without record, or deliberately disconnected.
- [ ] **Firmware and BIOS updates are included in the patch programme** — firmware is frequently excluded and is where some of the most persistent compromises live.
- [ ] **Network appliances, printers, and other embedded devices are patched too** — they run full operating systems, sit on the internal network, and are almost never in the patch plan.
- [ ] **Exceptions to patching have an owner, a compensating control, and an expiry date** — a machine that cannot be patched because of a vendor dependency should be isolated, not ignored.

## 6. Reconciliation against identity and finance data

- [ ] **The asset register is reconciled against the identity directory on a defined cycle** — every assigned asset should map to an active person, and every active person with an issued device should have one.
- [ ] **Assets assigned to disabled or deleted identities are surfaced as exceptions** — this is the single most productive reconciliation query in the whole practice.
- [ ] **The endpoint management platform, the endpoint detection platform, and the register are compared three ways** — a device present in two of the three is a coverage gap that no single tool will show you.
- [ ] **SaaS seat assignments are reconciled against active identities** — dormant seats are simultaneously an access risk and a cost.
- [ ] **The register is reconciled against finance's fixed asset and depreciation records** — divergence here signals either missing assets or overstated books, and finance will care about the second.
- [ ] **Discrepancies are investigated and closed with a recorded cause, not simply overwritten** — the cause tells you which process is leaking.
- [ ] **Reconciliation runs automatically and produces an exception report rather than requiring a person to compare spreadsheets** — a manual reconciliation happens twice and then stops.

## 7. Disposal and data sanitisation

- [ ] **A disposal procedure exists per media type and follows a recognised sanitisation standard** — clear, purge, or destroy chosen against the sensitivity of the data and the fate of the device.
- [ ] **Self-encrypting drives are sanitised by verified cryptographic erase, and the verification is recorded** — cryptographic erase is only as good as the proof that the key was actually destroyed.
- [ ] **Physical destruction is used where the data is sensitive or the drive cannot be verifiably erased** — a failed drive that cannot be written to also cannot be wiped.
- [ ] **Every disposed asset has a certificate of destruction or sanitisation linked to its serial number** — a certificate that lists a quantity rather than serial numbers proves nothing about a specific device.
- [ ] **Third-party disposal vendors are assessed, contracted, and audited** — chain of custody from your loading bay to the shredder is your liability, not theirs.
- [ ] **Devices with embedded storage are included: printers, multifunction devices, network gear, and cameras** — printer hard drives holding years of scanned documents are a recurring breach source.
- [ ] **Cloud storage, snapshots, and backups belonging to a decommissioned system are deleted too** — decommissioning the instance and leaving the volume snapshots is retained data with no owner.
- [ ] **The register is updated to disposed with the date, method, and evidence reference** — an asset that vanishes from the register has not been disposed of, it has been forgotten.

## 8. Governance, reporting, and audit readiness

- [ ] **The asset management policy is documented, approved, and reviewed on a schedule** — including what counts as an asset, which is more contentious than it sounds.
- [ ] **A single system of record is designated and other lists are subordinate to it** — two authoritative spreadsheets means zero authoritative spreadsheets.
- [ ] **Register accuracy is measured and reported as a metric, with a target** — sampled accuracy, reconciliation exception count, and unassigned-asset count.
- [ ] **Loss and theft are recorded, reported, and trended** — a rising loss rate is a process problem, and each loss may be a notifiable data event.
- [ ] **The register can answer a vulnerability question quickly** — which assets run this software at this version, answered in minutes rather than through a week of emails.
- [ ] **Access to modify the register is controlled and changes are logged** — an asset register anyone can edit without trace is not evidence.
- [ ] **Evidence for audit is retained for the period the relevant standard requires** — purchase records, sanitisation certificates, and reconciliation reports.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Hardware inventory | | | Pass / Pass with actions / Fail |
| Software inventory and discovery | | | Pass / Pass with actions / Fail |
| Ownership and lifecycle | | | Pass / Pass with actions / Fail |
| Licence compliance and true-up risk | | | Pass / Pass with actions / Fail |
| End-of-life and patching | | | Pass / Pass with actions / Fail |
| Reconciliation against identity and finance | | | Pass / Pass with actions / Fail |
| Disposal and data sanitisation | | | Pass / Pass with actions / Fail |
| Governance, reporting, and audit readiness | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with an owner, and treat any gap in sanitisation evidence as a potential data protection issue rather than an administrative one.

## Related checklists

- [Employee IT Offboarding](/docs/itsm/employee-it-offboarding/)
- [Employee IT Onboarding](/docs/itsm/employee-it-onboarding/)
- [Cloud Cost Optimization](/docs/cloud/cloud-cost-optimization/)
- [ISO 27001 ISMS](/docs/compliance/iso27001-isms/)
- [Vendor Security Assessment](/docs/compliance/vendor-security-assessment/)

## References

- [CIS Critical Security Controls](https://www.cisecurity.org/controls)
- [NIST SP 800-88 Rev. 1 — Guidelines for Media Sanitization](https://csrc.nist.gov/pubs/sp/800/88/r1/final)
- [ISO/IEC 27001 — Information Security Management](https://www.iso.org/standard/27001)
- [ITIL Service Management (Axelos)](https://www.axelos.com/certifications/itil-service-management)
- [NCSC Device Security Guidance](https://www.ncsc.gov.uk/collection/device-security-guidance)
