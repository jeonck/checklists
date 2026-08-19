---
title: "REST API Design"
description: "Verify an HTTP API is consistent, evolvable, and safe before its first external consumer depends on it."
icon: "api"
weight: 420
toc: true
tags: ["api", "http", "design", "backend"]
---

An HTTP API is the hardest kind of code to change, because the people who depend on it do not deploy when you do. Most of the cost of a bad API is paid years later, in compatibility shims and support tickets. Work through this before the first consumer outside your team integrates, and again before any version is declared stable.

{{< alert context="info" text="**Who runs this:** the owning team plus one reviewer who will have to consume the API. **When:** at design review, before the first endpoint ships, and again before the API is published externally." />}}

## 1. Resources and URL structure {#resources-and-url-structure}

- [ ] **Paths name resources, not actions** — `POST /orders/42/refunds` beats `POST /refundOrder`, because the resource model composes and the verb list does not.
- [ ] **Collections are plural and consistently cased** — pick one convention for multi-word segments and apply it everywhere; mixed `user_profiles` and `userProfiles` guarantees integration bugs.
- [ ] **Nesting stops at one level** — `/customers/{id}/orders` is useful, `/customers/{id}/orders/{id}/lines/{id}/taxes` forces callers to know an identifier hierarchy they should not need.
- [ ] **Identifiers in URLs are opaque to the client** — exposing an auto-incrementing primary key leaks your record count and invites enumeration.
- [ ] **A resource has exactly one canonical URL** — multiple aliases fragment caching, logging, and authorisation rules.
- [ ] **Genuinely non-CRUD operations are modelled as subordinate resources** — a state transition can be a `POST` to `/subscriptions/{id}/cancellations` rather than an RPC verb bolted onto the path.
- [ ] **Trailing slash, casing, and encoding behaviour is defined and consistent** — decide whether `/Orders` redirects or 404s, and make the router enforce it.

## 2. HTTP semantics {#http-semantics}

- [ ] **Method semantics are respected** — `GET` and `HEAD` never change state, `PUT` and `DELETE` are idempotent, and `POST` is the only method allowed to be neither safe nor idempotent.
- [ ] **`GET` requests carry no request body and no side effects** — intermediaries are entitled to retry, cache, and prefetch them.
- [ ] **Status codes distinguish the failure classes callers must handle differently** — 400 for a malformed request, 401 for missing or invalid authentication, 403 for authenticated but not permitted, 404 for absent, 409 for a state conflict, 422 for semantically invalid content, 429 for rate limiting.
- [ ] **201 responses carry a `Location` header** — the caller should not have to guess the URL of what it just created.
- [ ] **204 is used only when there is genuinely no body** — returning 200 with an empty body forces every client to special-case parsing.
- [ ] **5xx is reserved for server faults** — returning 500 for a validation failure trains callers to retry something that will never succeed and pollutes your error budget.
- [ ] **Long-running work returns 202 with a status resource** — a synchronous request that blocks for minutes will be killed by some proxy in the path.
- [ ] **Partial updates use a defined patch format** — `PATCH` with an ad-hoc merge dialect makes clearing a field to null indistinguishable from omitting it.

## 3. Request and response payloads {#request-and-response-payloads}

- [ ] **One media type is the default and it is negotiated properly** — honour `Accept`, and return 415 for an unsupported `Content-Type` rather than silently attempting to parse it.
- [ ] **Field naming is consistent across every endpoint** — one casing convention, and the same concept has the same field name everywhere.
- [ ] **Timestamps are RFC 3339 in UTC with an explicit offset** — a bare local datetime is the most common cross-team integration defect.
- [ ] **Monetary amounts carry an explicit currency and use a string or minor-unit integer** — a JSON number for money will be parsed as a float by somebody.
- [ ] **Enumerated values are documented and clients are told to tolerate unknown members** — otherwise adding a value becomes a breaking change.
- [ ] **Top-level responses are objects, never bare arrays** — an object leaves room to add pagination metadata later without breaking parsers.
- [ ] **Absent, null, and empty are distinguished deliberately** — document what each means for every optional field, because callers will infer something.
- [ ] **Request bodies have a maximum size enforced before parsing** — unbounded parsing is a trivially exploitable denial of service.

## 4. Errors {#errors}

- [ ] **Errors use a single machine-readable structure across the whole API** — RFC 9457 problem details is a reasonable default and saves every client writing a bespoke parser.
- [ ] **Every error carries a stable, documented error code** — clients must be able to branch on something other than the human-readable message, which you will want to reword.
- [ ] **Validation errors identify the offending field** — a 400 saying only "invalid request" turns integration into guesswork.
- [ ] **Error messages do not leak internals** — stack traces, SQL fragments, internal hostnames, and library versions are reconnaissance material.
- [ ] **Errors include a correlation identifier that also appears in your logs** — so a support ticket can be traced without asking the caller to reproduce.
- [ ] **Retryable and non-retryable failures are distinguishable** — pair 429 and 503 with `Retry-After` so well-behaved clients back off correctly.

```http
HTTP/1.1 422 Unprocessable Content
Content-Type: application/problem+json

{
  "type": "https://api.example.com/problems/invalid-field",
  "title": "Validation failed",
  "status": 422,
  "code": "order.line_items.empty",
  "detail": "An order must contain at least one line item.",
  "instance": "/orders",
  "trace_id": "3f9a1c8e"
}
```

## 5. Pagination, filtering, and sorting {#pagination-filtering-and-sorting}

- [ ] **Collection endpoints are paginated by default with a documented maximum page size** — an unpaginated collection is a time bomb attached to your slowest table.
- [ ] **Pagination uses a stable cursor rather than an offset** — with offsets, rows inserted or deleted mid-scan cause results to shift, duplicate, or be skipped entirely.
- [ ] **The cursor is opaque and its encoding is not part of the contract** — clients that decode it will break when you change the sort key.
- [ ] **A total count is either omitted or explicitly approximate** — an exact count over a large filtered set is often more expensive than the page itself.
- [ ] **Sorting is limited to indexed fields** — arbitrary sort parameters let a caller table-scan your production database.
- [ ] **Filter syntax is simple and closed** — a bespoke query language on a query string becomes an injection surface and an optimiser problem.
- [ ] **Deep pagination is bounded** — either cap the total traversable depth or require a narrower filter, rather than letting page 10,000 melt the database.

## 6. Versioning and evolution {#versioning-and-evolution}

- [ ] **The versioning strategy is decided and documented before launch** — retrofitting versioning onto a live API is far more expensive than choosing wrongly at the start.
- [ ] **What counts as a breaking change is written down** — removing or renaming a field, tightening validation, changing a status code, or adding a required request field all break existing callers.
- [ ] **Additive changes are safe because clients are documented as tolerant readers** — state explicitly that unknown response fields must be ignored.
- [ ] **Deprecation has a published policy with a minimum notice period** — and deprecated endpoints emit `Deprecation` and `Sunset` headers so callers can find them programmatically.
- [ ] **Usage per endpoint and per client is measured** — you cannot retire anything you cannot prove nobody is calling.
- [ ] **Old versions have a defined end-of-life date, not an open-ended one** — versions you never remove are versions you maintain forever.

{{< alert context="warning" text="**Common mistake:** treating a change as non-breaking because your own client still works. Tightening a validation rule, narrowing an enum, or making an optional response field disappear will break somebody, even though your tests stay green." />}}

## 7. Security and access control {#security-and-access-control}

- [ ] **Every endpoint requires authentication unless it is deliberately public** — the default in the router should be deny, with public routes as explicit exceptions.
- [ ] **Authorisation is enforced per object, not per route** — object-level authorisation failure is the most commonly exploited API vulnerability, and it always looks fine in a route table.
- [ ] **TLS is required and plaintext requests are rejected, not redirected** — a redirect has already exposed the credential.
- [ ] **Tokens are short-lived and scoped** — a long-lived token with full account scope turns a single leaked log line into a breach.
- [ ] **Rate limits exist per client and are communicated in response headers** — including the limit, the remaining quota, and the reset time.
- [ ] **Mass assignment is prevented by an explicit allowlist of writable fields** — binding a request body straight onto a model lets callers set `role` or `account_id`.
- [ ] **CORS policy names specific origins** — a wildcard combined with credentials is a mistake that survives review because it makes the browser errors stop.
- [ ] **Responses never include fields the caller is not entitled to see** — filtering in the client is not filtering.

## 8. Documentation and contract {#documentation-and-contract}

- [ ] **A machine-readable specification exists and is generated from or verified against the implementation** — hand-written OpenAPI drifts within one sprint.
- [ ] **Every endpoint documents its error responses, not just the success case** — the error contract is the part integrators actually need.
- [ ] **Examples are real and executable** — copy a request from the integration test suite rather than writing one by hand.
- [ ] **Authentication is documented end to end** — how to obtain a credential, how to refresh it, and what happens when it expires mid-request.
- [ ] **Rate limits, quotas, and pagination defaults are stated in the documentation** — not discovered in production by a caller who then files a support ticket.
- [ ] **A changelog records every change with its date and its compatibility impact.**

## 9. Operability {#operability}

- [ ] **Idempotency keys are supported on non-idempotent operations that create money movement or side effects** — clients will retry a timed-out `POST`, and without a key you will charge twice.
- [ ] **Cache behaviour is explicit on every response** — set `Cache-Control` deliberately, and use `ETag` with conditional requests where responses are expensive.
- [ ] **Request and response sizes, latency, and status code distribution are instrumented per endpoint** — aggregate API metrics hide the one endpoint that is failing.
- [ ] **Timeouts are defined at every layer and the client-facing timeout is documented** — so callers can set their own budget below yours rather than above it.
- [ ] **A load test has established the per-endpoint breaking point** — particularly for the endpoints that fan out to other services.
- [ ] **Contract tests run in CI against the published specification** — so a schema-breaking change fails the build rather than a customer's integration.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Resources and URL structure | | | Pass / Pass with actions / Fail |
| HTTP semantics | | | Pass / Pass with actions / Fail |
| Request and response payloads | | | Pass / Pass with actions / Fail |
| Errors | | | Pass / Pass with actions / Fail |
| Pagination, filtering, and sorting | | | Pass / Pass with actions / Fail |
| Versioning and evolution | | | Pass / Pass with actions / Fail |
| Security and access control | | | Pass / Pass with actions / Fail |
| Documentation and contract | | | Pass / Pass with actions / Fail |
| Operability | | | Pass / Pass with actions / Fail |

Record each "Pass with actions" as a dated ticket with an owner; anything unresolved in the versioning or security sections should block publication.

## Related checklists

- [Web Application Security](/docs/security/web-application-security/)
- [Code Review](/docs/development/code-review/)
- [Database Schema Migration](/docs/development/database-schema-migration/)
- [New Service Bootstrap](/docs/development/new-service-bootstrap/)
- [Observability](/docs/operations/observability/)

## References

- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 9457 — Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html)
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [MDN — HTTP reference](https://developer.mozilla.org/en-US/docs/Web/HTTP)
