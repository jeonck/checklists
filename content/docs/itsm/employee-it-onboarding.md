---
title: "Employee IT Onboarding"
description: "Verify a new joiner gets identity, device, and least-privilege access on time and without shortcuts."
icon: "person_add"
weight: 910
toc: true
tags: ["itsm", "identity", "onboarding", "access-control"]
---

Onboarding is the moment an organisation decides how much access a stranger gets, usually under time pressure and usually by copying somebody else's permissions. Done well it is boring: the account, the device, and the entitlements all exist before the person does, and each one is traceable to an approved request. Work through this in time order — before day one, day one, first week, first month — and treat the manager attestation at the end as the real completion criterion, not the laptop handover.

{{< alert context="info" text="**Who runs this:** IT service desk, with the hiring manager as requester and the identity owner as approver. **When:** started at least five working days before the start date, closed out within 30 days." />}}

## 1. Before day one: request, approval, and role

- [ ] **A joiner request exists in the ticketing system with the hiring manager named as requester** — verbal or chat-based onboarding requests leave no approval record when an auditor asks who authorised the access.
- [ ] **The start date, employment type, and contract end date are recorded** — contractors and fixed-term staff need an expiry date on the account from the outset, not a reminder that nobody sets.
- [ ] **The role maps to a defined access profile, not to another person** — copying a colleague's permissions is how one over-privileged account quietly becomes twenty.
- [ ] **Any access outside the standard profile is itemised and separately approved** — production access, finance systems, and customer data need a named approver per system, not a blanket yes.
- [ ] **HR has confirmed the hire in the system of record before provisioning starts** — provisioning from an email alone is the pretext used in most onboarding fraud attempts.
- [ ] **Background or right-to-work checks required before access are confirmed complete or explicitly deferred with a date** — track the deferral rather than letting it disappear.
- [ ] **Hardware is ordered with enough lead time and a delivery address is confirmed** — for remote joiners, confirm someone will be there to sign for it.

## 2. Identity account creation

- [ ] **The account is created in the authoritative identity provider and nowhere else** — accounts created directly in a downstream application escape joiner-mover-leaver processes entirely.
- [ ] **The username follows the standard convention and does not collide with a former employee's identifier** — recycled identifiers inherit stale group memberships and mail routing.
- [ ] **The account carries the manager, department, employment type, and start date as attributes** — these attributes drive group membership and later reconciliation, so an empty manager field breaks the whole chain.
- [ ] **The account is disabled or time-gated until the start date** — an active account sitting unused for two weeks is an unmonitored foothold.
- [ ] **The initial credential is single-use and delivered out of band** — a temporary password sent in the same channel as the username is one compromised inbox away from an account takeover.
- [ ] **Licence assignment is driven by group membership, not by hand** — manual licence assignment is the most common cause of both cost drift and day-one blockers.

## 3. Multi-factor enrolment and credential hygiene

- [ ] **Multi-factor authentication is enforced by policy before the first sign-in, not offered afterwards** — a grace period is a window in which a leaked temporary password is enough.
- [ ] **The enrolment is supervised or bound to a verified channel** — unsupervised first-time enrolment lets an attacker who has the temporary password register their own authenticator.
- [ ] **Phishing-resistant factors are the default where supported** — passkeys or hardware security keys, with one-time codes as a fallback only for systems that cannot do better.
- [ ] **SMS is not used as a primary factor for privileged roles** — number porting attacks are cheap and well documented.
- [ ] **A documented account recovery path exists that does not rely on the service desk trusting a voice on the phone** — helpdesk social engineering is now a mainstream intrusion route.
- [ ] **The password manager is provisioned and the joiner is shown how to use it on day one** — if you do not give people a place to put credentials, they will use a spreadsheet.

## 4. Device provisioning and management enrolment

- [ ] **The device is enrolled in the mobile device management platform before handover** — a device that is enrolled only when the user gets round to it is a device that is never enrolled.
- [ ] **Full-disk encryption is enabled and the recovery key is escrowed to the management platform** — an encrypted laptop with a recovery key on a sticky note is theatre.
- [ ] **The endpoint detection agent is installed, reporting, and visible in the console** — check the console, not the installer log; agents that install and never check in are common.
- [ ] **The device is patched to current baseline before handover** — new hardware routinely ships with firmware and OS versions that are months behind.
- [ ] **Screen lock, idle timeout, and disk-level protections match the device policy** — verify by looking at the compliance state in the console rather than trusting the profile assignment.
- [ ] **Local administrator rights are not granted by default** — where a role genuinely needs them, use a time-bound elevation mechanism and record the justification.
- [ ] **The asset is recorded in the asset register against the joiner with serial number and assignment date** — this is the record you will rely on at offboarding.
- [ ] **Personal devices used for work are covered by a documented policy and enrolled at least to the level that permits selective wipe** — decide this before the person needs mail on their phone, not after.

## 5. Access grants and least privilege

- [ ] **Every entitlement is granted through a group, not directly to the user** — direct grants are invisible to access reviews and survive role changes.
- [ ] **Production, customer data, and financial systems are excluded from the default profile** — these should require an explicit request even for engineers who will obviously need them later.
- [ ] **Standing privileged access is replaced by just-in-time elevation where the platform supports it** — a permanent administrator account is a permanent target.
- [ ] **Access to shared mailboxes and team drives is granted by membership, not by sharing a password** — shared credentials destroy attribution and cannot be revoked for one person.
- [ ] **Third-party SaaS accounts go through single sign-on wherever the vendor supports it** — every non-SSO account is an account you will have to remember to disable manually at offboarding.
- [ ] **Source control, cloud console, and CI access are granted at the lowest tier that lets the person do the job in week one** — write access to production infrastructure is not a week-one requirement.
- [ ] **Any exception to least privilege has a named approver, a reason, and a review date** — undated exceptions become permanent.

{{< alert context="warning" text="**Common failure:** granting the new joiner the same access as a long-tenured colleague because it is faster. That colleague has accumulated years of entitlements from roles they no longer hold, and you have just cloned all of them into a fresh account." />}}

## 6. Day one: handover and first sign-in

- [ ] **Identity is verified at handover for remote joiners** — a video call against the photo ID already held by HR, or collection in person; couriered laptops have been intercepted.
- [ ] **The joiner completes first sign-in, multi-factor enrolment, and a password change while support is available** — day-one failures that go unresolved become week-one shadow IT.
- [ ] **The acceptable use policy and device policy are acknowledged and the acknowledgement is stored** — an unsigned policy is unenforceable.
- [ ] **The joiner knows how to reach the service desk and how to report a suspected phishing message** — including the reporting path on a phone when the laptop is the thing that is compromised.
- [ ] **Communication tools, calendar, and directory profile work and the joiner appears in the org chart** — a joiner missing from the directory gets excluded from access reviews as well as from meetings.
- [ ] **Any day-one access that has not arrived is logged as a ticket with an owner** — not left as an informal promise from whoever is doing the handover.

## 7. First week: training and baseline

- [ ] **Security awareness training is completed, including phishing, social engineering, and credential handling** — schedule it in the first week while it is still treated as part of the job rather than an interruption.
- [ ] **Data classification and handling rules are covered with examples from the joiner's actual role** — generic training does not tell a support agent what to do with a customer's exported data.
- [ ] **Role-specific compliance training is assigned where the role requires it** — payment data, health data, or export-controlled work each carry their own obligations.
- [ ] **The joiner has run through the tools they will actually use, with someone watching** — this surfaces missing entitlements faster than any access report.
- [ ] **Training completion is recorded against the person, with a due date and an escalation for non-completion** — a training platform nobody chases is a training platform nobody uses.

## 8. First month: verification and attestation

- [ ] **The manager attests that the granted access matches the role, item by item** — this is the control that catches over-provisioning while it is still cheap to fix.
- [ ] **Access that was requested but never used in the first month is flagged for removal** — first-month usage data is the cleanest signal you will ever get about what a role actually needs.
- [ ] **Temporary or day-one elevated rights have expired and are confirmed gone** — check the effective permissions, not the ticket status.
- [ ] **The asset register, identity directory, and HR record agree on this person** — a mismatch found now is a five-minute fix; found at offboarding it is an incident.
- [ ] **The onboarding ticket is closed with all sub-tasks resolved and the evidence attached** — screenshots of compliance state and approval records, not a free-text note saying done.
- [ ] **For contractors, the account expiry date is set and tested** — confirm the directory will actually disable the account on that date rather than merely displaying it.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Request, approval, and role | | | Pass / Pass with actions / Fail |
| Identity account creation | | | Pass / Pass with actions / Fail |
| Multi-factor enrolment and credential hygiene | | | Pass / Pass with actions / Fail |
| Device provisioning and management enrolment | | | Pass / Pass with actions / Fail |
| Access grants and least privilege | | | Pass / Pass with actions / Fail |
| Day one handover and first sign-in | | | Pass / Pass with actions / Fail |
| First week training and baseline | | | Pass / Pass with actions / Fail |
| First month verification and attestation | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with an owner, and do not close the onboarding record until the manager attestation in section 8 is complete.

## Related checklists

- [Employee IT Offboarding](/docs/itsm/employee-it-offboarding/)
- [IT Asset Management](/docs/itsm/it-asset-management/)
- [Secrets Management](/docs/security/secrets-management/)
- [ISO 27001 ISMS](/docs/compliance/iso27001-isms/)
- [Cloud Security](/docs/security/cloud-security/)

## References

- [CIS Critical Security Controls](https://www.cisecurity.org/controls)
- [NIST SP 800-63B — Digital Identity Guidelines: Authentication](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [Microsoft Entra ID Governance — Lifecycle Workflows](https://learn.microsoft.com/en-us/entra/id-governance/what-are-lifecycle-workflows)
- [AWS IAM Security Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [NCSC Device Security Guidance](https://www.ncsc.gov.uk/collection/device-security-guidance)
