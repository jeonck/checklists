---
title: "Employee IT Offboarding"
description: "Verify a leaver loses every route back in, fast, including the accounts that never touched single sign-on."
icon: "person_remove"
weight: 920
toc: true
tags: ["itsm", "identity", "offboarding", "access-control"]
---

Offboarding is judged on two axes: speed and completeness. Speed matters because the risk window is the gap between the person's last moment of goodwill and the moment their last credential stops working. Completeness matters because attackers and disgruntled leavers do not use the front door you remembered to lock — they use the forgotten local account, the personal access token, the shared credential nobody rotated. Run this the same way every time, whether the departure is amicable or not.

{{< alert context="info" text="**Who runs this:** IT service desk and the identity owner, triggered by HR, with the leaving manager responsible for data handover. **When:** revocation steps execute on the last working day at the agreed hour; for involuntary departures, before the conversation happens." />}}

## 1. Trigger, timing, and classification {#trigger-timing-and-classification}

- [ ] **HR is the trigger and the leaving date and time are recorded in the system of record** — offboarding driven by word of mouth is offboarding that happens late or not at all.
- [ ] **The departure is classified as voluntary, involuntary, or immediate-risk** — an involuntary departure is revoked before notification, not after, and that decision must be made in advance.
- [ ] **The exact revocation time is agreed with the manager and HR** — cutting access at 09:00 on someone's last day when they are mid-handover creates pressure to reopen accounts informally.
- [ ] **The leaver's full inventory is assembled before the day: accounts, devices, entitlements, group memberships, and delegated rights** — you cannot revoke what you have not enumerated, and enumeration takes longer than revocation.
- [ ] **Privileged, administrative, and production access is identified and prioritised** — these come first in the revocation sequence, not last.
- [ ] **A single named owner is accountable for the whole offboarding record** — split ownership between HR, IT, and the manager is the reason offboardings stall at 80 percent complete.

## 2. Session and token revocation in the identity provider {#session-and-token-revocation-in-the-identity-provider}

- [ ] **Active sessions and refresh tokens are explicitly revoked in the identity provider** — not just a password reset; a live OAuth session or refresh token survives a password change and can keep issuing new access tokens for weeks.
- [ ] **The account is disabled rather than deleted at first** — deletion destroys audit trail, breaks file ownership, and cannot be undone if you need to investigate something later.
- [ ] **Registered multi-factor authenticators and passkeys are removed from the account** — a retained authenticator plus a self-service password reset is a complete route back in.
- [ ] **Device and browser trust or remembered-device state is cleared** — trusted-device flags can bypass the multi-factor prompt entirely.
- [ ] **Persistent application-specific passwords and legacy authentication paths are revoked** — legacy protocols frequently ignore modern conditional access policy.
- [ ] **Conditional access or sign-in risk policies are checked for exclusions naming this user** — individual exclusions added for a past troubleshooting session outlive the reason for them.
- [ ] **Revocation is verified by attempting the sign-in flows the leaver actually used** — the console showing disabled is a claim; a failed authentication is evidence.

## 3. Accounts and credentials outside single sign-on {#accounts-and-credentials-outside-single-sign-on}

- [ ] **Local accounts on servers, network devices, databases, and appliances are enumerated and disabled** — these never appeared in the identity provider and will not appear in any SSO-based report.
- [ ] **Personal access tokens, API keys, and deploy keys issued by the leaver are revoked in every platform** — source control, cloud providers, CI systems, and monitoring tools all allow long-lived tokens that keep working after the human account is disabled.
- [ ] **Service accounts and automation the leaver created are re-owned, not orphaned** — an unowned service account is an unmonitored account with production credentials.
- [ ] **SSH keys, client certificates, and VPN profiles associated with the leaver are removed and the revocation is confirmed at the server side** — removing the key from the laptop does nothing if the public key is still in an authorised keys file.
- [ ] **Personal email forwarding rules, mail delegation, and auto-forwarding on the leaver's mailbox are inspected and removed** — a forwarding rule set before departure exfiltrates mail long after the account is gone.
- [ ] **Third-party OAuth grants the leaver authorised against corporate data are reviewed and revoked** — the app keeps its grant even when the user account is disabled in some platforms.

{{< alert context="danger" text="**This is the gap that bites.** Most offboardings handle the SSO account cleanly and miss two things: accounts that were never behind SSO (local admin accounts, vendor portals, database logins, that one SaaS tool bought on a card), and long-lived API tokens the leaver issued themselves. Both survive account disablement. Enumerate them explicitly and prove each one is dead." />}}

## 4. Shared and privileged credential rotation {#shared-and-privileged-credential-rotation}

- [ ] **Every shared credential the leaver had access to is rotated, not merely un-shared** — knowledge cannot be revoked; removing a vault share does not un-know the password.
- [ ] **The password vault is queried for everything shared with the leaver, including items shared to groups they belonged to** — group-based sharing is the part people forget.
- [ ] **Root, break-glass, and emergency accounts are rotated if the leaver had or could have had the credential** — including the sealed envelope in the safe.
- [ ] **Shared cloud provider and hosting credentials are rotated and the rotation is confirmed by a successful login with the new value** — a rotation that broke automation and got rolled back is not a rotation.
- [ ] **Signing keys, code-signing certificates, and release credentials the leaver held are rotated or revoked** — these have the highest blast radius of anything on this list.
- [ ] **Rotation is prioritised by blast radius and completed within a defined window** — for a departing administrator, that window is hours, not the end of the month.
- [ ] **Where rotation must be deferred for operational reasons, the exception is written down with a date and an owner** — deferred rotations are otherwise never done.

## 5. Device return, wipe, and endpoint state {#device-return-wipe-and-endpoint-state}

- [ ] **All assigned devices are listed from the asset register and each one is accounted for** — laptop, phone, tablet, security keys, dongles, and any home networking equipment.
- [ ] **Returned devices are confirmed received against serial number, not against model name** — two identical laptops are a reconciliation error waiting to happen.
- [ ] **Remote wipe or lock is issued for devices not returned by the agreed date, and the command is confirmed executed** — a queued wipe on a device that never checks in has not wiped anything.
- [ ] **Personal devices with corporate data receive a selective wipe of the managed apps and profiles** — and the leaver is told this will happen before it happens.
- [ ] **Disk encryption recovery keys for returned devices are retained until the device is reimaged, then rotated with the new assignment** — you will need the key if the device comes back locked.
- [ ] **Local data on returned devices is preserved if there is any prospect of investigation or litigation hold** — reimaging first destroys the only copy of evidence.
- [ ] **The asset register is updated to unassigned or in-stock the same day** — a register that lags reality is worse than no register, because people trust it.

## 6. Data handover, mailbox, and file ownership {#data-handover-mailbox-and-file-ownership}

- [ ] **The manager identifies the business-critical data the leaver owned before the last day** — documents, dashboards, scripts, scheduled jobs, and the undocumented process only they ran.
- [ ] **Files in personal drives are transferred to a named successor or a team location** — personal drive content is deleted on a schedule after account removal in most platforms, and it goes silently.
- [ ] **Mailbox delegation or a shared-mailbox conversion is set up for the agreed period with a defined end date** — indefinite mailbox delegation gives a colleague permanent access to a former employee's correspondence.
- [ ] **An auto-reply or mail routing rule directs correspondents to the right person** — and it does not disclose the reason for departure.
- [ ] **Ownership of calendars, recurring meetings, distribution lists, and group memberships is reassigned** — an orphaned distribution list owner means nobody can manage its membership.
- [ ] **Repositories, pipelines, cloud resources, and scheduled automation owned by the leaver are reassigned to a team, not to an individual** — otherwise you repeat this exercise at the successor's departure.
- [ ] **Any legal hold or retention requirement is applied to the mailbox and files before the account is removed** — retention applied after deletion is not retention.

## 7. Physical access, facilities, and third parties {#physical-access-facilities-and-third-parties}

- [ ] **Badge, fob, and door codes are deactivated and the deactivation is confirmed in the access control system** — collecting the badge is not the same as disabling it.
- [ ] **Shared door codes, alarm codes, and safe combinations known to the leaver are changed** — the same logic as shared credentials applies to physical access.
- [ ] **Data centre, colocation, and building visitor authorisation lists are updated with the provider** — third-party facility lists are maintained by the third party and will not update themselves.
- [ ] **Corporate cards, procurement authority, and expense system access are revoked with finance** — spending authority tends to outlive system access.
- [ ] **Vendor and supplier portals where the leaver was the named contact or administrator are transitioned** — including domain registrars, certificate authorities, and payment processors, where a lost administrator can take weeks to recover.
- [ ] **External communities, partner slack connections, and customer-facing accounts are removed** — a leaver still present in a shared customer channel is both a security and a commercial problem.

## 8. SaaS estate sweep {#saas-estate-sweep}

- [ ] **The SaaS inventory is walked application by application, not sampled** — the applications people forget are exactly the ones bought outside procurement.
- [ ] **Applications discovered through expense reports and browser or network telemetry are included** — shadow IT does not appear in the official application list by definition.
- [ ] **Licences are reclaimed and the seat count is reduced where the contract allows** — offboarding is the cheapest cost optimisation available and it is routinely skipped.
- [ ] **Applications where the leaver was the sole administrator are re-administered before their account is disabled** — order matters here; disable first and you may be locked out of your own tenant.
- [ ] **Data stored only in a leaver-owned SaaS workspace is exported or transferred** — personal-tier accounts used for work delete their content on cancellation.
- [ ] **Each application is marked complete with the date and the person who verified it** — a partially swept estate looks identical to a fully swept one unless you record it.

## 9. Post-offboarding audit and closure {#post-offboarding-audit-and-closure}

- [ ] **An access report is run 24 to 72 hours after the revocation date and shows no successful authentication by the leaver** — this is the check that catches the account you missed.
- [ ] **Sign-in logs are reviewed for activity in the days before departure that looks like bulk download or mass sharing** — unusual export volumes shortly before a resignation are worth a conversation.
- [ ] **The account is deleted or archived only after the retention period defined by policy** — and file ownership has already been transferred, because deletion may cascade.
- [ ] **The offboarding record is closed with evidence for each section, not a single completion tick** — screenshots, ticket references, and rotation confirmations.
- [ ] **Time-to-revocation is measured and tracked as a metric** — if you do not measure it, the median quietly drifts from hours to weeks.
- [ ] **Anything that could not be revoked is escalated rather than closed** — an unrecoverable vendor account with the leaver as administrator is a risk item for the register, not a footnote in a ticket.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Trigger, timing, and classification | | | Pass / Pass with actions / Fail |
| Session and token revocation | | | Pass / Pass with actions / Fail |
| Accounts and credentials outside SSO | | | Pass / Pass with actions / Fail |
| Shared and privileged credential rotation | | | Pass / Pass with actions / Fail |
| Device return, wipe, and endpoint state | | | Pass / Pass with actions / Fail |
| Data handover, mailbox, and file ownership | | | Pass / Pass with actions / Fail |
| Physical access, facilities, and third parties | | | Pass / Pass with actions / Fail |
| SaaS estate sweep | | | Pass / Pass with actions / Fail |
| Post-offboarding audit and closure | | | Pass / Pass with actions / Fail |

Any section that is not a clear Pass must be escalated to the identity owner the same day, since a partial offboarding leaves an active route into the estate.

## Related checklists

- [Employee IT Onboarding](/docs/itsm/employee-it-onboarding/)
- [Secrets Management](/docs/security/secrets-management/)
- [IT Asset Management](/docs/itsm/it-asset-management/)
- [Incident Response](/docs/security/incident-response/)
- [SOC 2 Audit Readiness](/docs/compliance/soc2-audit-readiness/)

## References

- [CIS Critical Security Controls](https://www.cisecurity.org/controls)
- [Microsoft Entra — Revoke user access in an emergency](https://learn.microsoft.com/en-us/entra/identity/users/users-revoke-access)
- [AWS IAM Security Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [NIST SP 800-53 Rev. 5 — Security and Privacy Controls](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [ISO/IEC 27001 — Information Security Management](https://www.iso.org/standard/27001)
