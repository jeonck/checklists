---
title: "Security Incident Response"
description: "Verify you can detect, contain, investigate, and communicate a security incident without improvising under pressure."
icon: "emergency"
weight: 150
toc: true
tags: ["incident-response", "forensics", "breach", "soc"]
---

A security incident is an operational incident with lawyers, regulators, and an adversary who reacts to what you do. The decisions that matter most — whether to isolate a host, when to rotate credentials, who tells the regulator — are terrible decisions to make for the first time at 2am. This checklist covers both readiness and the live response, in the order events actually unfold: prepare, detect, contain, eradicate, recover, and learn.

{{< alert context="info" text="**Who runs this:** the security incident commander, with the affected service owner, legal, and communications. **When:** review readiness quarterly; work the response sections live during an incident and again during a tabletop exercise." />}}

## 1. Preparation and readiness {#preparation-and-readiness}

- [ ] **An incident response plan exists, names roles rather than individuals, and is stored where it is reachable when the corporate network is down** — a plan on the compromised wiki is not a plan.
- [ ] **Severity levels are defined with concrete examples and mapped to response times** — the difference between a single phished mailbox and a domain administrator compromise must not be argued about live.
- [ ] **The incident commander role is separate from the hands-on investigator role** — one person cannot both direct a response and debug a host.
- [ ] **An out-of-band communication channel is provisioned and tested** — assume email and chat are readable by the attacker until proven otherwise.
- [ ] **Legal, privacy, communications, and executive contacts are on-call reachable, with deputies.**
- [ ] **Retainers with external forensics and legal counsel are signed in advance** — negotiating a contract is not a step you want inside your notification deadline.
- [ ] **Cyber insurance requirements are understood, particularly any obligation to notify the insurer before engaging responders.**
- [ ] **Tabletop exercises have been run in the last twelve months, including at least one scenario where a privileged administrator account is compromised.**

## 2. Detection and triage {#detection-and-triage}

- [ ] **Telemetry covers identity, endpoint, network, cloud control plane, and application layers** — an attacker who only touches the cloud API leaves no endpoint trace.
- [ ] **Log retention is long enough to investigate a dwell time measured in months, and logs are stored where an attacker in the production account cannot alter them.**
- [ ] **Detections exist for the high-value behaviours** — impossible-travel logins, MFA fatigue and push bombing, new privileged role assignment, mass download, and disabled logging.
- [ ] **There is one obvious way for anyone, including an external researcher, to report a suspected incident** — a monitored address plus a published security contact.
- [ ] **Triage assigns a severity and an incident commander within a defined time from first alert.**
- [ ] **Every incident gets a ticket and a dedicated channel from the moment it is declared** — investigation done in direct messages cannot be reconstructed afterwards.

## 3. Containment {#containment}

- [ ] **Volatile evidence is captured before containment where feasible** — memory, active connections, and running processes disappear when a host is powered off.
- [ ] **Containment actions are decided deliberately rather than reflexively** — alerting the attacker too early can trigger destructive action or a pivot you cannot follow.
- [ ] **Compromised accounts are disabled and their sessions and refresh tokens revoked** — a password reset alone leaves existing tokens working.
- [ ] **Affected hosts are isolated at the network level rather than shut down** — isolation preserves memory and the ability to observe.
- [ ] **Credentials, keys, and tokens the attacker could have reached are rotated, with an ordered list so the most privileged go first.**
- [ ] **Persistence mechanisms are hunted before the environment is declared contained** — new OAuth grants, mail forwarding rules, added SSH keys, scheduled tasks, and new IAM users are the standard set.
- [ ] **Containment steps are logged with timestamps and the person who performed them** — this record is the backbone of the later timeline.

{{< alert context="danger" text="**Do not restore from backup or rebuild before scope is understood.** Rebuilding a host destroys the evidence needed to determine whether the same access exists elsewhere, and an attacker with valid credentials will simply return to the rebuilt system." />}}

## 4. Investigation and scope {#investigation-and-scope}

- [ ] **A single timeline is maintained in one place, in UTC, and updated as facts are confirmed** — separate people keeping separate notes produces contradictory reporting to regulators.
- [ ] **Facts are recorded separately from hypotheses** — early theories in an incident are wrong more often than they are right.
- [ ] **Initial access, lateral movement, privilege escalation, and exfiltration paths are each investigated rather than assumed.**
- [ ] **The question of what data was accessed is answered with evidence** — the absence of exfiltration logs is not evidence that nothing left.
- [ ] **Chain of custody is maintained for any evidence that may be needed legally** — hashes, acquisition times, and handler names.
- [ ] **Indicators of compromise are extracted and swept across the whole estate, not only the known-affected systems.**
- [ ] **Third parties and vendors in the blast radius are identified early** — your incident may be their incident, and their contract may impose a notification clock.

## 5. Communication {#communication}

- [ ] **One person owns external communication and everyone else refers enquiries to them** — inconsistent statements create legal exposure that outlives the incident.
- [ ] **Regulatory notification deadlines are identified in the first hours** — 72 hours under GDPR from becoming aware, with other regimes running shorter or longer clocks.
- [ ] **Customer notification content is drafted from confirmed facts and reviewed by legal before release.**
- [ ] **Internal updates go out on a fixed cadence even when there is nothing new** — silence generates rumour and pulls responders into answering the same question repeatedly.
- [ ] **Staff know not to discuss the incident externally, and know where to send questions.**
- [ ] **Law enforcement engagement is a conscious decision made with counsel, not an afterthought.**

## 6. Eradication and recovery {#eradication-and-recovery}

- [ ] **Root cause is confirmed before rebuild, so the same entry point is not reinstated.**
- [ ] **Systems are rebuilt from known-good images rather than cleaned in place** — you cannot prove a compromised host is clean.
- [ ] **Backups are validated as clean before restoration** — restoring a backup taken after the initial intrusion reintroduces the attacker.
- [ ] **All credentials in the blast radius are rotated before systems are returned to service, including service accounts and signing keys.**
- [ ] **Recovery is staged with heightened monitoring on the restored systems** — reintrusion attempts commonly follow within days.
- [ ] **A defined set of criteria must be met before the incident is declared closed, and someone with authority declares it.**

## 7. Post-incident learning {#post-incident-learning}

- [ ] **A blameless post-incident review happens within two weeks, while memory is fresh.**
- [ ] **The review covers detection and response performance, not only the technical vulnerability** — how long until detection, until containment, until notification.
- [ ] **Actions are specific, owned, and dated, and are tracked in the normal engineering backlog rather than a separate document nobody opens.**
- [ ] **New detections are written for the behaviours that went unnoticed** — this is the highest-value output of any incident.
- [ ] **The response plan itself is updated with what did not work.**
- [ ] **A summary is shared with the wider organisation, redacted as needed** — incidents are the most effective security training material you will ever have.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Preparation and readiness | | | Pass / Pass with actions / Fail |
| Detection and triage | | | Pass / Pass with actions / Fail |
| Containment | | | Pass / Pass with actions / Fail |
| Investigation and scope | | | Pass / Pass with actions / Fail |
| Communication | | | Pass / Pass with actions / Fail |
| Eradication and recovery | | | Pass / Pass with actions / Fail |
| Post-incident learning | | | Pass / Pass with actions / Fail |

For a live incident, keep this sign-off with the incident record; for a readiness review, date it and set the next review before closing.

## Related checklists

- [Incident Management](/docs/operations/incident-management/)
- [Postmortem](/docs/operations/postmortem/)
- [Cloud Security Posture](/docs/security/cloud-security/)
- [Backup and Recovery](/docs/operations/backup-and-recovery/)
- [Disaster Recovery Drill](/docs/itsm/disaster-recovery-drill/)

## References

- [NIST SP 800-61 Computer Security Incident Handling Guide](https://csrc.nist.gov/pubs/sp/800/61/r2/final)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [NCSC — Incident Management Collection](https://www.ncsc.gov.uk/collection/incident-management)
- [FIRST — Standards and Frameworks](https://www.first.org/standards/frameworks/)
- [Google SRE Book — Managing Incidents](https://sre.google/sre-book/managing-incidents/)
