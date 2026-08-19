---
title: "Code Review"
description: "Verify a pull request is correct, readable, and safe to merge before you approve it."
icon: "rate_review"
weight: 410
toc: true
tags: ["code-review", "quality", "collaboration", "engineering"]
---

Code review is the cheapest place to catch a defect and the most expensive place to argue about brace style. This checklist is what a reviewer should hold in their head while reading a change: does it do what it claims, will it survive contact with production, and can the next person understand it. It also covers the author's side, because most slow reviews are caused by a pull request that was never made reviewable.

{{< alert context="info" text="**Who runs this:** the reviewer, with the author completing section 1 before requesting review. **When:** on every pull request that changes production code, configuration, or infrastructure." />}}

## 1. Before review is requested

- [ ] **The change does one thing** — a refactor and a behaviour change in the same pull request means the reviewer cannot tell which diff lines are the risky ones.
- [ ] **The description states the intent, not the diff** — what problem is being solved and what alternative was rejected; the reviewer can already read what changed.
- [ ] **The change is under roughly 400 lines of meaningful diff** — defect detection falls off sharply above that, and reviewers start rubber-stamping.
- [ ] **Generated files, lockfiles, and formatting-only changes are in separate commits** — so the reviewer can skip them without skipping the real change.
- [ ] **The author has read their own diff first** — leftover debug statements, commented-out code, and stray `TODO` markers are the author's job to catch, not the reviewer's.
- [ ] **CI is green before review is requested** — asking a human to review code that does not compile wastes the scarcest resource on the team.
- [ ] **Linked to the issue or ticket it resolves** — future archaeologists need the why, and the ticket usually holds it.

## 2. Correctness

- [ ] **The code does what the description says it does** — read the change against the stated intent, not against what you assume it should do.
- [ ] **Boundary conditions are handled** — empty collections, single-element collections, zero, negative numbers, maximum values, and the first and last iteration.
- [ ] **Error paths are as considered as the happy path** — every caught exception either handles the error meaningfully or re-raises it; swallowing an exception into a log line hides failures until they compound.
- [ ] **Null, nil, or absent values are handled deliberately** — an optional value that is dereferenced without a check is a production incident waiting for the right input.
- [ ] **Concurrency assumptions are explicit** — shared mutable state, non-atomic read-modify-write sequences, and check-then-act races are the defects reviewers most often miss.
- [ ] **Time, timezone, and locale handling is correct** — store and compute in UTC, format at the edge, and never assume a day is 24 hours.
- [ ] **Floating-point arithmetic is not used for money** — use integer minor units or a decimal type, and check that no rounding is done implicitly.
- [ ] **Resources are released on every path** — file handles, connections, locks, and cursors, including when an exception unwinds the stack.

## 3. Design and structure

- [ ] **The change fits the existing architecture, or the divergence is deliberate and explained** — a second way of doing something that already has a way is a maintenance tax.
- [ ] **Abstractions are introduced because there are concrete callers, not in anticipation of them** — speculative generality is harder to remove than duplication.
- [ ] **Public interfaces are as narrow as they can be** — everything exported becomes a contract someone will depend on.
- [ ] **Dependency direction is sane** — business logic should not import the web framework, and a domain module should not know about the database driver.
- [ ] **New third-party dependencies are justified** — check licence, maintenance status, transitive dependency count, and whether the standard library already does it.
- [ ] **The code is not duplicating a utility that already exists in the repository** — search before you accept a new helper.
- [ ] **Backwards compatibility of any shared contract is preserved** — API responses, event payloads, database columns, and queue message formats all have consumers you cannot deploy atomically with.

## 4. Readability and naming

- [ ] **Names say what the thing is, not what type it is** — `retryBudget` beats `intVal`, and a boolean named `flag` tells the reader nothing.
- [ ] **Comments explain why, not what** — a comment restating the code rots; a comment recording the reason for a non-obvious decision earns its keep.
- [ ] **Functions fit the reader's working memory** — if you have to scroll to see the whole function, the reviewer cannot verify it.
- [ ] **Nesting depth is kept shallow** — early returns and guard clauses beat a pyramid of conditionals.
- [ ] **Magic numbers and strings are named constants** — especially timeouts, retry counts, and limits, which someone will need to tune under pressure.
- [ ] **The change matches the file's existing style** — consistency inside a file matters more than the reviewer's personal preference.

## 5. Tests

- [ ] **The tests would fail without the change** — the surest check is to mentally revert the production code and ask which test breaks.
- [ ] **A bug fix comes with a regression test that reproduces the original bug** — otherwise nothing stops it returning.
- [ ] **Tests assert on behaviour, not on implementation detail** — a test that breaks on every refactor will be deleted rather than fixed.
- [ ] **Edge cases identified in section 2 have corresponding tests** — the reviewer should be able to point at a test for each one.
- [ ] **Tests are deterministic** — no reliance on wall-clock time, real network calls, random ordering, or `sleep` for synchronisation.
- [ ] **Test data does not contain real personal data or real credentials** — fixtures leak into logs, screenshots, and public repositories.
- [ ] **Failure messages are informative** — assert on values, not on booleans, so a red build says what was expected and what was received.

## 6. Security and data handling

- [ ] **All external input is validated at the boundary** — including values from other internal services, which are not more trustworthy than the internet, just differently compromised.
- [ ] **Queries are parameterised** — string-concatenated SQL, shell commands, or template expressions are the classic injection route.
- [ ] **Authorisation is checked on the object, not just on the route** — a valid session for user A must not be able to fetch user B's record by changing an identifier.
- [ ] **Secrets are not introduced into the repository** — including test configuration, sample `.env` files, and fixtures.
- [ ] **New logging does not emit credentials, tokens, or personal data** — check any log line that dumps a request, response, or whole object.
- [ ] **Output is encoded for its destination** — HTML escaping, JSON encoding, and shell quoting are context-specific and cannot be done once at input time.
- [ ] **Cryptographic primitives come from a vetted library with modern defaults** — a hand-rolled token, hash, or comparison is a finding, not a preference.

{{< alert context="danger" text="**Do not approve** a change that adds an unparameterised query, disables certificate verification, or removes an authorisation check with the explanation that it is only for a test environment. These reach production far more often than anyone expects." />}}

## 7. Performance and operational impact

- [ ] **No query is issued inside a loop over results** — the N+1 pattern passes review at ten rows and fails at ten thousand.
- [ ] **New queries are supported by an index, and the query plan has been checked** — not assumed from the column name.
- [ ] **Unbounded reads are paginated or streamed** — loading a whole table into memory works until the table grows.
- [ ] **New outbound calls have timeouts and a defined failure behaviour** — and the reviewer can say what happens to the user when that call fails.
- [ ] **Caching changes state ownership deliberately** — the invalidation rule and the staleness the product accepts should be written down, not implied.
- [ ] **The change is observable** — a new failure mode with no metric, log, or trace attached is invisible until a customer reports it.

## 8. Review conduct

- [ ] **Comments distinguish blocking from non-blocking** — prefix optional suggestions clearly so the author knows what actually holds up the merge.
- [ ] **Feedback addresses the code, not the author** — phrase findings as questions about the code's behaviour rather than judgements about the person.
- [ ] **The reviewer offers a concrete alternative when rejecting an approach** — "this is wrong" is not actionable feedback.
- [ ] **Disagreements that survive two rounds move to a synchronous conversation** — asynchronous escalation between two people is how a one-day review becomes a one-week review.
- [ ] **Approval means the reviewer would be comfortable being paged for this code** — that is the standard, not "I found nothing obviously wrong in five minutes".
- [ ] **Review turnaround is within one business day** — a stale branch diverges, and the author has already lost the context needed to respond well.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Before review is requested | | | Pass / Pass with actions / Fail |
| Correctness | | | Pass / Pass with actions / Fail |
| Design and structure | | | Pass / Pass with actions / Fail |
| Readability and naming | | | Pass / Pass with actions / Fail |
| Tests | | | Pass / Pass with actions / Fail |
| Security and data handling | | | Pass / Pass with actions / Fail |
| Performance and operational impact | | | Pass / Pass with actions / Fail |
| Review conduct | | | Pass / Pass with actions / Fail |

Anything recorded as "Pass with actions" should become a follow-up ticket linked from the pull request before it is merged.

## Related checklists

- [Security Code Review](/docs/security/security-code-review/)
- [REST API Design](/docs/development/rest-api-design/)
- [Database Schema Migration](/docs/development/database-schema-migration/)
- [CI/CD Pipeline](/docs/devops/cicd-pipeline/)
- [Production Readiness Review](/docs/devops/production-readiness/)

## References

- [Google Engineering Practices — Code Review Developer Guide](https://google.github.io/eng-practices/review/)
- [OWASP Code Review Guide](https://owasp.org/www-project-code-review-guide/)
- [OWASP Top Ten](https://owasp.org/www-project-top-ten/)
- [Conventional Comments](https://conventionalcomments.org/)
