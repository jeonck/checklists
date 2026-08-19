---
title: "Web Application Security Review"
description: "Verify a web application defends against the injection, access control, and session flaws that actually get exploited."
icon: "security"
weight: 110
toc: true
tags: ["appsec", "owasp", "web", "review"]
---

Most web applications are not broken into through exotic zero-days. They are broken into through a missing authorisation check on an object ID, a session that never expires, or a file upload that lands in a directory the web server will happily execute. This review walks the request path from the browser to the database and back, in the order an attacker would probe it.

{{< alert context="info" text="**Who runs this:** an application security engineer together with a developer who knows the codebase. **When:** before the first public release, after any change to authentication or authorisation, and at least annually for anything internet-facing." />}}

## 1. Authentication

- [ ] **Passwords are stored with a memory-hard hash** — bcrypt, scrypt, or Argon2id with tuned parameters; SHA-256 with a salt is not adequate against modern GPU cracking.
- [ ] **Credential stuffing is throttled per account and per source** — rate limit on the username as well as the IP, because attackers spread a single password across many accounts from many addresses.
- [ ] **Multi-factor authentication is available and enforced for privileged accounts** — TOTP or WebAuthn; SMS is a fallback, not a control you rely on for admins.
- [ ] **Account enumeration is not possible through login, registration, or password reset** — identical responses and identical timing whether or not the account exists.
- [ ] **Password reset tokens are single-use, random, and short-lived** — bound to the account, invalidated on use, and expiring within an hour.
- [ ] **The reset flow does not trust the Host header** — a reset link built from an attacker-controlled `Host` sends the token to the attacker.
- [ ] **Default and seeded credentials are removed from every environment** — including the demo admin account someone added for a sales pitch.

## 2. Session management

- [ ] **Session cookies carry HttpOnly, Secure, and SameSite=Lax or Strict** — HttpOnly blocks theft via XSS, Secure blocks it over plaintext, SameSite blunts CSRF.
- [ ] **The session identifier is rotated on login and on privilege change** — otherwise an attacker who fixes a victim's session ID before login inherits the authenticated session.
- [ ] **Sessions expire on both idle timeout and absolute lifetime** — an idle timeout alone lets a stolen token live forever if it is used periodically.
- [ ] **Logout invalidates the session server-side** — deleting the cookie in the browser does nothing to a token an attacker already copied.
- [ ] **JWTs, if used, pin the algorithm server-side and validate issuer, audience, and expiry** — accepting the `alg` header from the token is how `none` and key-confusion attacks work.
- [ ] **There is a way to revoke a session or token before it expires** — needed the moment a laptop is lost or an account is compromised.

## 3. Authorisation and access control

- [ ] **Every request is authorised server-side against the authenticated principal** — hiding a button in the UI is not access control.
- [ ] **Object-level authorisation is checked on every object reference** — the classic IDOR is `GET /invoices/1042` returning someone else's invoice because only authentication was checked.
- [ ] **Authorisation is enforced in a central, hard-to-bypass layer** — a per-controller `if` statement will eventually be forgotten on a new endpoint.
- [ ] **The default for a new endpoint is deny** — allow-listing what is public beats blocklisting what is protected.
- [ ] **Horizontal and vertical privilege escalation have been tested explicitly** — log in as a low-privilege user and replay an admin request with their session.
- [ ] **Multi-tenant queries are scoped by tenant at the data layer** — a missing `WHERE tenant_id = ?` is a cross-customer data breach, not a bug.
- [ ] **Mass assignment is prevented** — bind request bodies to an explicit allow-list of fields, or a user updates their own `role` field.

{{< alert context="danger" text="**Blocking:** broken object-level authorisation is the single most commonly exploited web flaw and is invisible to most automated scanners. Test it by hand, with two accounts, before release." />}}

## 4. Input handling and injection

- [ ] **All database access uses parameterised queries or a well-used ORM** — string concatenation into SQL is still the fastest route to full data loss.
- [ ] **Dynamic query fragments such as sort columns are mapped through an allow-list** — parameter binding cannot protect an identifier or an `ORDER BY` clause.
- [ ] **OS command execution avoids the shell and passes arguments as an array** — and no user input reaches a command line at all where it can be avoided.
- [ ] **Templating engines are used in their auto-escaping mode with no raw interpolation of user data** — server-side template injection escalates straight to code execution.
- [ ] **Deserialisation of untrusted data is avoided, or restricted to a safe format with a type allow-list** — native object deserialisation is remote code execution waiting for input.
- [ ] **Input validation is positive** — validate against expected type, length, format, and range rather than trying to filter known-bad strings.
- [ ] **XML parsers have external entity resolution disabled** — XXE turns a document upload into a file-read and internal-network probe.

## 5. Output encoding and browser-side defences

- [ ] **Output is contextually encoded at the point of rendering** — HTML body, attribute, JavaScript, URL, and CSS contexts each need different escaping.
- [ ] **Any use of a raw-HTML sink is reviewed and sanitised with a maintained library** — `innerHTML`, `dangerouslySetInnerHTML`, and their equivalents are where DOM XSS lives.
- [ ] **A Content Security Policy is deployed without unsafe-inline or unsafe-eval** — nonce or hash based, so an injected script has nowhere to execute.
- [ ] **State-changing requests require an anti-CSRF token or are restricted to same-site requests** — SameSite cookies help but do not cover every browser or every flow.
- [ ] **CORS does not reflect arbitrary origins and does not combine a wildcard with credentials** — reflecting `Origin` with `Allow-Credentials: true` is equivalent to no same-origin policy at all.
- [ ] **Security headers are set** — HSTS with a long max-age, `X-Content-Type-Options: nosniff`, and a restrictive `Referrer-Policy` and `Permissions-Policy`.
- [ ] **Clickjacking is prevented** — `frame-ancestors` in the CSP, with `X-Frame-Options` for legacy clients.

## 6. Business logic and abuse resistance

- [ ] **Server-side price, quantity, and discount values are authoritative** — never trust an amount that came back from the client.
- [ ] **Workflow steps cannot be skipped or replayed** — verify that jumping straight to the confirmation endpoint fails.
- [ ] **Race conditions on limited resources are handled with locking or atomic operations** — parallel requests are how a single-use coupon gets redeemed fifty times.
- [ ] **Expensive endpoints are rate limited and quota bounded** — search, export, and report generation are cheap to request and expensive to serve.
- [ ] **Server-side requests to user-supplied URLs are blocked or proxied through an allow-list** — SSRF into cloud metadata endpoints is a standard path to credential theft.

## 7. Data protection and privacy

- [ ] **All traffic is HTTPS with modern TLS and HTTP redirected permanently** — including internal service-to-service calls.
- [ ] **Sensitive data is not placed in URLs** — query strings end up in access logs, referrer headers, and browser history.
- [ ] **Personal and sensitive fields are encrypted at rest where the threat model demands it** — with keys held outside the database.
- [ ] **Error responses are generic and stack traces are never returned to the client** — detailed errors map your framework, versions, and file layout for an attacker.
- [ ] **Caching headers prevent sensitive responses being stored by browsers or intermediaries.**
- [ ] **Uploaded files are validated by content, stored outside the web root, and served with a fixed content type** — and never with a user-controlled filename or extension.

## 8. Dependencies and configuration

- [ ] **A software bill of materials is generated and dependencies are scanned on every build** — with a policy for how quickly a critical finding must be fixed.
- [ ] **Framework and server configurations are hardened for production** — debug mode off, directory listing off, admin consoles unreachable from the internet.
- [ ] **Client-side third-party scripts are inventoried and justified** — every tag manager script runs with full access to your DOM and cookies.
- [ ] **Secrets are loaded from a secret manager, not from the repository or the container image** — verify with a history scan, not just a look at the current tree.
- [ ] **Security-relevant events are logged with enough context to investigate** — authentication outcomes, authorisation failures, and privileged actions, with no credentials in the log.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Authentication | | | Pass / Pass with actions / Fail |
| Session management | | | Pass / Pass with actions / Fail |
| Authorisation and access control | | | Pass / Pass with actions / Fail |
| Input handling and injection | | | Pass / Pass with actions / Fail |
| Output encoding and browser defences | | | Pass / Pass with actions / Fail |
| Business logic and abuse resistance | | | Pass / Pass with actions / Fail |
| Data protection and privacy | | | Pass / Pass with actions / Fail |
| Dependencies and configuration | | | Pass / Pass with actions / Fail |

Every finding that is not a clean pass needs a severity, an owner, and a fix date agreed before release.

## Related checklists

- [Security Code Review](/docs/security/security-code-review/)
- [Penetration Test Readiness](/docs/security/penetration-test-readiness/)
- [Secrets Management](/docs/security/secrets-management/)
- [REST API Design](/docs/development/rest-api-design/)
- [Production Readiness Review](/docs/devops/production-readiness/)

## References

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Application Security Verification Standard](https://owasp.org/www-project-application-security-verification-standard/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/)
