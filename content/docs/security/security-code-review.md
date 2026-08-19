---
title: "Security Code Review"
description: "Verify a change introduces no new security weakness before it is merged, by reading the code with an attacker in mind."
icon: "policy"
weight: 130
toc: true
tags: ["appsec", "code-review", "sdlc", "static-analysis"]
---

A security code review is a normal code review with one extra question asked of every line: what happens if the input is hostile? It is cheaper than a penetration test and catches a different class of problem — the missing check, the unsafe default, the trust boundary crossed without anyone noticing. Use this list when reviewing a change that touches authentication, authorisation, data handling, cryptography, or anything that parses untrusted input.

{{< alert context="info" text="**Who runs this:** the reviewer on a pull request, escalating to a security engineer for changes to authentication, authorisation, cryptography, or payment flows. **When:** before merge, not before release." />}}

## 1. Scoping the review {#scoping-the-review}

- [ ] **The purpose of the change is understood before reading the diff** — you cannot spot a missing check if you do not know what the code is supposed to enforce.
- [ ] **Trust boundaries crossed by the change are identified** — network edge, tenant boundary, privilege boundary, and process boundary each carry different obligations.
- [ ] **Files changed are checked against a security-sensitive path list** — auth modules, middleware, query builders, deserialisers, crypto helpers, and CI configuration warrant a deeper read.
- [ ] **The threat model or design note is consulted for anything new** — a review is not the right place to first discover the feature accepts uploads from anonymous users.
- [ ] **Automated findings are triaged before the human read** — so reviewer attention goes to the logic that tools cannot reason about.

## 2. Input handling and trust {#input-handling-and-trust}

- [ ] **Every external input is identified and validated at the boundary** — request bodies, headers, cookies, path parameters, webhook payloads, queue messages, and file contents.
- [ ] **Validation is allow-list based and applied server-side** — client-side validation is a usability feature and provides no security value.
- [ ] **Data crossing a trust boundary is re-validated even if it was validated upstream** — an internal service call is not a guarantee of well-formed input.
- [ ] **Canonicalisation happens before validation, not after** — path traversal and Unicode tricks defeat checks applied to the raw string.
- [ ] **Numeric inputs are bounded and integer overflow or truncation is considered** — especially on quantities, offsets, and sizes.
- [ ] **Untrusted input never reaches a shell, a query string, a template, a deserialiser, or a URL fetch without an explicit safe construction.**

## 3. Authentication and authorisation logic {#authentication-and-authorisation-logic}

- [ ] **Every new endpoint, route, message handler, or GraphQL resolver has an explicit authorisation decision** — check the framework's default, and confirm it is deny rather than allow.
- [ ] **Authorisation is applied to the object, not just the operation** — confirm the record being loaded actually belongs to the requesting principal or tenant.
- [ ] **The identity used for the decision comes from the verified session or token, never from a request parameter** — a `user_id` in the body is an attacker-supplied value.
- [ ] **Comparisons for secrets, tokens, and signatures use a constant-time function** — an ordinary equality check leaks the value byte by byte under timing analysis.
- [ ] **New administrative or internal endpoints are not reachable from the public edge** — verify with routing configuration, not with an assumption about the load balancer.
- [ ] **Changes to roles or permission definitions are reviewed for unintended grants** — adding a permission to a shared role changes access for everyone holding it.

## 4. Data handling and storage {#data-handling-and-storage}

- [ ] **Database access uses parameterised queries throughout the change** — including any dynamically assembled fragment.
- [ ] **Personal or sensitive data added by the change is classified and its storage location is intentional** — new columns, new caches, new analytics events, and new log lines all count.
- [ ] **Nothing sensitive is written to logs, traces, metrics labels, or error messages** — search the diff for logging of request bodies, headers, tokens, and exception objects.
- [ ] **Sensitive values are not placed in URLs, query strings, or redirect targets.**
- [ ] **Serialised responses expose only the intended fields** — returning a whole model object leaks whatever is added to it later.
- [ ] **Temporary files, caches, and scratch directories are created with restrictive permissions and cleaned up.**

## 5. Cryptography and secrets {#cryptography-and-secrets}

- [ ] **No secret, key, token, or certificate is present in the diff** — including test fixtures, sample configuration, and comments.
- [ ] **Cryptographic operations use a maintained high-level library rather than primitives assembled by hand** — a custom construction is a defect until proven otherwise.
- [ ] **Randomness for tokens, IDs, and keys comes from a cryptographically secure source** — not from the default pseudo-random generator.
- [ ] **Algorithms and key sizes meet current guidance, and deprecated ones are absent** — MD5, SHA-1, DES, RC4, and ECB mode have no place in new code.
- [ ] **Initialisation vectors and nonces are never reused with the same key, and authenticated encryption is used by default.**
- [ ] **TLS verification is not disabled anywhere in the change** — a disabled certificate check added to fix a staging problem invariably ships to production.

## 6. Dependencies and supply chain {#dependencies-and-supply-chain}

- [ ] **New dependencies are justified, actively maintained, and reviewed for their own transitive weight** — every dependency is code you now run.
- [ ] **Dependency versions are pinned and resolved through a lockfile committed to the repository.**
- [ ] **Package names are checked against typosquats and against internal package names** — dependency confusion works because a public package can shadow a private one.
- [ ] **Changes to build files, CI configuration, and pipeline scripts get the same scrutiny as application code** — the pipeline usually holds more privilege than the app.
- [ ] **Post-install scripts and build-time code execution introduced by a dependency are noticed and questioned.**

## 7. Error handling, concurrency, and resource use {#error-handling-concurrency-and-resource-use}

- [ ] **Failures default to denying access** — an exception in an authorisation check must not fall through to allow.
- [ ] **Errors returned to the caller are generic; details go to the log with a correlation ID.**
- [ ] **Shared mutable state introduced by the change is protected, and check-then-act sequences are made atomic** — race conditions on balances, quotas, and single-use tokens are exploitable, not theoretical.
- [ ] **Loops, recursion, and allocations driven by user input are bounded** — an unbounded expansion is a denial of service with a one-line request.
- [ ] **Resources are released on every path, including error paths.**

## 8. Testing and verification {#testing-and-verification}

- [ ] **Security-relevant behaviour has a negative test** — a test proving the unauthorised caller is rejected is worth more than one proving the authorised caller is allowed.
- [ ] **Static analysis and secret scanning run on the change and their new findings are resolved or explicitly accepted with a reason.**
- [ ] **Any suppression or baseline entry added to a scanner has a comment explaining why and a review date** — suppressions are how findings disappear permanently.
- [ ] **A reviewer has actually exercised the risky path manually where automation cannot** — authorisation bypasses are found by trying, not by reading alone.
- [ ] **Findings are recorded with a severity and an owner rather than resolved verbally in a comment thread.**
- [ ] **Test fixtures and seed data introduced by the change contain no real personal data** — production dumps used as test data leak through repositories and CI logs.
- [ ] **Feature flags gating the new behaviour default to off and their removal is scheduled** — a half-reviewed path left permanently enabled is the one that gets exploited.

{{< alert context="warning" text="**Common mistake:** approving a change because the automated scans are green. Static analysis finds injection patterns and hardcoded secrets well, and finds missing authorisation almost never. The logic review is the part that cannot be delegated to a tool." />}}

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Scoping the review | | | Pass / Pass with actions / Fail |
| Input handling and trust | | | Pass / Pass with actions / Fail |
| Authentication and authorisation logic | | | Pass / Pass with actions / Fail |
| Data handling and storage | | | Pass / Pass with actions / Fail |
| Cryptography and secrets | | | Pass / Pass with actions / Fail |
| Dependencies and supply chain | | | Pass / Pass with actions / Fail |
| Error handling, concurrency, and resource use | | | Pass / Pass with actions / Fail |
| Testing and verification | | | Pass / Pass with actions / Fail |

Link the sign-off to the pull request so the review is auditable after the branch is deleted.

## Related checklists

- [Code Review](/docs/development/code-review/)
- [Web Application Security Review](/docs/security/web-application-security/)
- [Secrets Management](/docs/security/secrets-management/)
- [CI/CD Pipeline](/docs/devops/cicd-pipeline/)
- [Container Image](/docs/devops/container-image/)

## References

- [OWASP Code Review Guide](https://owasp.org/www-project-code-review-guide/)
- [OWASP Application Security Verification Standard](https://owasp.org/www-project-application-security-verification-standard/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [MITRE CWE Top 25 Most Dangerous Software Weaknesses](https://cwe.mitre.org/top25/)
- [NIST Secure Software Development Framework (SP 800-218)](https://csrc.nist.gov/pubs/sp/800/218/final)
