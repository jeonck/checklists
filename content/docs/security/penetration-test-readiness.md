---
title: "Penetration Test Readiness"
description: "Verify scope, access, environment, and authorisation are settled before a penetration test starts, so the engagement finds real issues."
icon: "bug_report"
weight: 160
toc: true
tags: ["pentest", "assurance", "scoping", "appsec"]
---

A penetration test is expensive per day and the days are easily wasted. Testers regularly spend the first two of five days waiting for credentials, discovering the environment does not match production, or re-finding issues an internal scanner already reported. This checklist gets the engagement ready so the testers spend their time on the things only a human can find: chained logic flaws, authorisation gaps, and abuse of the features you are proud of.

{{< alert context="info" text="**Who runs this:** the engagement owner on the customer side, with the service owner and the testing provider. **When:** start four weeks before the test window and finish the access items at least five working days before day one." />}}

## 1. Objectives and scope {#objectives-and-scope}

- [ ] **The question the test must answer is written down** — proving compliance, validating a new architecture, and simulating a determined attacker lead to three very different engagements.
- [ ] **In-scope assets are listed precisely** — domains, IP ranges, application URLs, API base paths, mobile application builds, and cloud account identifiers.
- [ ] **Out-of-scope assets are listed explicitly, with a reason** — shared SaaS platforms and third-party hosted components usually cannot be tested without their provider's consent.
- [ ] **The test type is agreed** — black box, grey box with credentials, or white box with source code; grey box finds substantially more per day for most applications.
- [ ] **The starting position of the simulated attacker is defined** — unauthenticated internet user, low-privilege customer, compromised employee laptop, or malicious insider.
- [ ] **Destructive testing boundaries are agreed in writing** — whether denial of service, social engineering, phishing, and physical access are permitted.
- [ ] **Success criteria and the deliverable format are agreed up front** — report structure, severity scale, retest inclusion, and whether an attestation letter is needed.

## 2. Authorisation and legal {#authorisation-and-legal}

- [ ] **A signed authorisation to test exists, naming the assets, the dates, and the testing team** — this is the document that keeps testing lawful; without it the activity is unauthorised access.
- [ ] **The cloud provider's testing policy has been checked and any required notification submitted** — the major providers permit customer testing of your own resources within published rules.
- [ ] **Third-party hosting, SaaS, and managed service providers have consented where their infrastructure is in scope.**
- [ ] **Non-disclosure and data handling terms cover any production or personal data the testers may encounter.**
- [ ] **Rules of engagement include a stop condition and the phone number that triggers it** — a real outage during a test needs a route to halt immediately.
- [ ] **Testers' source IP addresses are documented and shared with the operations team.**

## 3. Environment preparation {#environment-preparation}

- [ ] **The environment under test matches production in version, configuration, and defences** — a test against staging with the WAF disabled produces findings that do not apply to production.
- [ ] **Any deliberate difference from production is documented and given to the testers** — otherwise they report the difference as a finding.
- [ ] **Test data is realistic in shape and volume, and contains no real personal data unless that is a deliberate, approved decision.**
- [ ] **The environment is stable and not being deployed to mid-test** — agree a change freeze or a deployment notification protocol for the test window.
- [ ] **Rate limiting, IP blocking, and bot protection are configured deliberately** — either allow-list the testers so they can reach the application logic, or leave the controls on because testing them is the point, but decide which.
- [ ] **Monitoring and backups are confirmed working before day one** — tests do occasionally break things.

## 4. Access and credentials {#access-and-credentials}

- [ ] **Test accounts exist at every privilege level in scope, at least two per level** — cross-account authorisation testing needs two peers, and this is where the highest-severity findings usually come from.
- [ ] **Credentials are delivered securely and verified working by the testers before the window opens** — a login that fails on day one costs a full day.
- [ ] **Multi-factor authentication on test accounts is either exempted or provisioned with a shared enrolment the testers can actually use.**
- [ ] **API documentation, an OpenAPI or GraphQL schema, and a working example request are supplied** — undocumented endpoints go untested.
- [ ] **A Postman collection, HAR file, or recorded walkthrough of the main user journeys is provided** — testers cannot attack a workflow they have not seen.
- [ ] **VPN or bastion access is provisioned and tested for anything not reachable from the internet.**
- [ ] **Source code, architecture diagrams, and threat models are shared where the engagement is white box.**

{{< alert context="warning" text="**The most common cause of a wasted engagement is credentials that do not work on day one.** Have the testers log in to every account, at every privilege level, through every access path, a week before the test begins." />}}

## 5. Internal preparation {#internal-preparation}

- [ ] **Known issues, open findings, and recent scanner output are shared with the testers** — paying a specialist to rediscover what your scanner already told you is a poor use of the budget.
- [ ] **Low-hanging fruit is fixed before the test** — missing security headers and outdated dependencies fill a report without teaching you anything.
- [ ] **Blue team, SOC, and on-call engineers are told whether this is an announced test or a detection exercise** — if it is announced, tell them, or you will burn a night of on-call time.
- [ ] **A technical point of contact is available throughout the window to unblock testers and answer environment questions.**
- [ ] **Any monitoring alerts expected to fire during testing are pre-annotated so genuine incidents remain visible.**
- [ ] **Customer-facing support teams know a test is running** — so a spike in odd account activity is not escalated as fraud.
- [ ] **A budget and approval route for emergency remediation exists before the test** — a critical finding on day two needs an engineer freed up on day three.

## 6. During the test {#during-the-test}

- [ ] **Critical findings are reported immediately rather than held for the final report** — an exploitable path to customer data should not wait two weeks for a PDF.
- [ ] **A short daily check-in covers progress, blockers, and coverage** — this is where you discover a whole module has not been reached.
- [ ] **Coverage is tracked against the agreed scope, not only findings count** — an area nobody reached is a gap, not a clean result.
- [ ] **Any accidental impact on availability or data is reported and recorded by both sides.**
- [ ] **Testers record enough reproduction detail for a developer to reproduce without further contact** — request, payload, account used, and timestamp.

## 7. Findings, remediation, and closure {#findings-remediation-and-closure}

- [ ] **Every finding has a reproducible proof of concept and a clearly stated business impact** — severity without impact leads to endless argument during triage.
- [ ] **Severities are agreed jointly, taking your compensating controls and exposure into account** — a critical on an internal-only system may not outrank a medium on the public login.
- [ ] **Findings are converted into tracked engineering tickets with owners and dates within a week of the report.**
- [ ] **Root causes are addressed rather than individual instances** — one missing authorisation check usually means the pattern is missing everywhere.
- [ ] **A retest is scheduled and its scope is agreed while the engagement is still fresh.**
- [ ] **The report is stored with restricted access and a retention date** — it is a precise map of how to attack you.
- [ ] **Lessons feed back into the development lifecycle** — new test cases, new static analysis rules, and updated code review guidance.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Objectives and scope | | | Pass / Pass with actions / Fail |
| Authorisation and legal | | | Pass / Pass with actions / Fail |
| Environment preparation | | | Pass / Pass with actions / Fail |
| Access and credentials | | | Pass / Pass with actions / Fail |
| Internal preparation | | | Pass / Pass with actions / Fail |
| During the test | | | Pass / Pass with actions / Fail |
| Findings, remediation, and closure | | | Pass / Pass with actions / Fail |

The first five rows must be signed off before the test window opens; the last two are completed after the engagement ends.

## Related checklists

- [Web Application Security Review](/docs/security/web-application-security/)
- [Security Code Review](/docs/security/security-code-review/)
- [Cloud Security Posture](/docs/security/cloud-security/)
- [SOC 2 Audit Readiness](/docs/compliance/soc2-audit-readiness/)
- [Vendor Security Assessment](/docs/compliance/vendor-security-assessment/)

## References

- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [PTES Penetration Testing Execution Standard](http://www.pentest-standard.org/)
- [NIST SP 800-115 Technical Guide to Information Security Testing and Assessment](https://csrc.nist.gov/pubs/sp/800/115/final)
- [AWS Penetration Testing Policy](https://aws.amazon.com/security/penetration-testing/)
- [NCSC — Penetration Testing Guidance](https://www.ncsc.gov.uk/guidance/penetration-testing)
