---
title: "Load Balancer Configuration"
description: "Check health checks, timeouts, draining, and TLS on a load balancer before it fronts real traffic."
icon: "balance"
weight: 740
toc: true
tags: ["load-balancer", "networking", "reliability"]
---

A load balancer is the component most likely to be blamed for an outage and least likely to have caused it on its own. Its defaults are chosen to be safe for a generic workload, and almost every one of them — health check interval, idle timeout, draining period, stickiness — interacts with something specific about your backend. This checklist works through those interactions, with particular attention to the relationship between the load balancer's timeouts and the backend's own.

{{< alert context="info" text="**Who runs this:** the service owner together with the platform or network team that owns the load balancer. **When:** before the load balancer takes production traffic, and again after any change to backend timeouts or deployment strategy." />}}

## 1. Topology and listeners

- [ ] **The load balancer type matches the traffic** — a layer 7 proxy for HTTP where you need path routing and header manipulation, a layer 4 balancer where you need protocol transparency or client IP preservation without headers.
- [ ] **Listeners cover every port and protocol clients actually use** — including plain HTTP for the redirect to HTTPS, and any secondary admin or metrics port that should explicitly not be exposed.
- [ ] **The load balancer is deployed across at least two availability zones** — a single-zone balancer makes the balancer itself the single point of failure it was meant to remove.
- [ ] **Security groups or firewall rules allow only the load balancer to reach the backends** — otherwise clients can bypass the balancer, the WAF, and the rate limits by addressing a backend directly.
- [ ] **HTTP/2 or HTTP/3 support is a deliberate choice on both the client side and the backend side** — protocol downgrade between the two halves is normal and fine, but should be intentional rather than discovered.
- [ ] **The client's real IP address reaches the application** — via X-Forwarded-For or the PROXY protocol, and the application must trust that header only from the balancer, or clients can spoof their own address.

## 2. Health checks

- [ ] **The health check hits an endpoint that reflects the service's real ability to serve** — a static 200 from the web server proves the process is running and nothing else.
- [ ] **The health check does not fail because a downstream dependency is down** — if every backend deregisters when a shared database is slow, you convert a degraded service into a total outage with zero healthy targets.
- [ ] **Health check interval, timeout, and thresholds are set together and understood as a detection time** — interval multiplied by unhealthy threshold is how long a dead backend keeps receiving requests, and that number should be a deliberate one.
- [ ] **The health check timeout is shorter than the interval** — overlapping checks pile up and produce confusing, delayed state transitions.
- [ ] **Healthy and unhealthy thresholds are asymmetric** — fail fast to remove a bad backend, recover slowly to prevent a flapping backend from being repeatedly re-added mid-recovery.
- [ ] **Health check load is accounted for in backend capacity** — a two-second interval across many balancer nodes and many targets is real traffic, and a health check that queries a database multiplies it.
- [ ] **Health check failures are logged with enough detail to diagnose** — status code and response time per target, not merely a healthy or unhealthy flag.
- [ ] **A deliberately failed backend has been tested end to end** — stop a backend and observe both that traffic stops reaching it and how long that took.

## 3. Timeouts

- [ ] **The load balancer idle timeout is shorter than the backend keep-alive timeout** — this is the most common misconfiguration in the whole list: if the backend closes an idle pooled connection first, the balancer will send a request onto a connection that is closing and the client sees a sporadic 502.
- [ ] **The backend keep-alive timeout is set explicitly rather than left at the framework default** — many application servers default to a few seconds, well below typical balancer idle timeouts.
- [ ] **The request timeout at the balancer is longer than the slowest legitimate request** — including file uploads, report generation, and any long-poll or streaming endpoint, or those requests get cut off mid-flight and appear as client errors.
- [ ] **Timeouts are consistent along the whole chain** — CDN, load balancer, ingress, service mesh sidecar, and application, each layer at least as patient as the one behind it, or an inner retry will outlive an outer timeout.
- [ ] **WebSocket and server-sent event endpoints have their own timeout treatment** — a 60 second idle timeout silently kills long-lived connections and the application sees only unexplained disconnects.
- [ ] **Connection timeout to the backend is short** — establishing a TCP connection to a healthy backend should take milliseconds, so a long connect timeout only delays failover.
- [ ] **Retry behaviour at the balancer is understood and limited** — retrying non-idempotent requests duplicates writes, and retrying against an overloaded backend fleet amplifies the overload.

```nginx
# upstream keepalive must outlive the proxy's idle timeout, not the reverse
keepalive_timeout 75s;      # backend
proxy_read_timeout 60s;     # balancer/proxy side, deliberately shorter
```

{{< alert context="warning" text="**Common mistake:** intermittent 502 errors under low traffic almost always mean the backend keep-alive timeout is shorter than the load balancer idle timeout. Raise the backend value above the balancer value, rather than adding a retry to paper over the race." />}}

## 4. Connection draining and deployments

- [ ] **Connection draining or deregistration delay is enabled** — without it, terminating an instance drops every in-flight request on it, which shows up as a burst of errors on every deploy.
- [ ] **The drain period is longer than the longest normal request** — draining for 30 seconds while a report endpoint takes 90 truncates those requests anyway.
- [ ] **The application handles SIGTERM by failing its health check first, then draining** — quitting immediately on SIGTERM makes the drain period pointless, because the balancer is still sending new requests during it.
- [ ] **The orchestrator's termination grace period exceeds the drain period** — otherwise the platform kills the process while the balancer is still politely draining it.
- [ ] **Rolling deployments respect minimum healthy target counts** — replacing targets faster than new ones become healthy leaves the remaining backends carrying the whole load.
- [ ] **A deployment has been observed at the balancer's own metrics** — watch for a spike in 5xx or reset connections during a deploy in a lower environment, since that is where draining bugs are visible.
- [ ] **Scale-in events drain the same way as deployments** — autoscaling terminations are the forgotten path, and they drop connections if only the deploy pipeline was tested.

## 5. Session affinity

- [ ] **Whether the application needs sticky sessions at all is confirmed** — stickiness is almost always a workaround for in-memory session state, and moving state to a shared store removes an entire class of problems.
- [ ] **Where stickiness is used, its cookie attributes are correct** — Secure, HttpOnly, and an appropriate SameSite value, since a balancer-issued cookie is still a cookie in the user's browser.
- [ ] **Stickiness duration is bounded** — indefinite affinity produces persistent imbalance where a few backends carry most of the long-lived users.
- [ ] **The application tolerates losing affinity** — a backend will eventually be replaced, and the user must be re-routed without losing their session or seeing an error.
- [ ] **Load distribution is measured with stickiness enabled** — check requests per backend, since affinity plus long sessions routinely produces a two-to-one skew that pure round robin would not.
- [ ] **Sticky sessions and connection draining are tested together** — draining a backend that holds affinity for many users must move them cleanly rather than dropping them.

## 6. TLS termination and re-encryption

- [ ] **The termination model is a deliberate decision** — terminate at the edge for simplicity, or pass through where the backend must see the original certificate or client certificate.
- [ ] **Where traffic is re-encrypted to the backend, the backend certificate is actually validated** — re-encryption with validation disabled provides encryption but no authentication, so it does not defend against a compromised network path.
- [ ] **The backend certificate's SAN includes the name the balancer uses to address it** — addressing a backend by IP address while presenting a hostname certificate fails validation, which is why teams disable it, which is the previous item.
- [ ] **The certificate on the listener covers every hostname served through it** — including any hostname added later via a new routing rule, since the rule and the certificate are separate objects.
- [ ] **The TLS policy on the listener is a current, maintained one** — TLS 1.2 minimum, forward-secret suites only, and re-verified after any platform-managed policy update.
- [ ] **Certificate renewal on the balancer is automated and monitored** — the balancer is where hand-uploaded certificates accumulate, and where their expiry is least visible.
- [ ] **HTTP to HTTPS redirection happens at the balancer, not the application** — the application should never be asked to serve anything over plain HTTP.

## 7. Traffic management and resilience

- [ ] **The balancing algorithm suits the workload** — least outstanding requests handles heterogeneous request costs far better than round robin, which is only fair when every request costs the same.
- [ ] **Cross-zone balancing is enabled, or its absence is a deliberate choice** — with it disabled and uneven target counts per zone, a zone with two backends receives the same share as a zone with ten, and each of the two carries five times the load.
- [ ] **The cost of cross-zone traffic is understood where the provider charges for it** — this is the legitimate reason to disable it, and it requires keeping target counts balanced per zone.
- [ ] **Outlier detection or passive health checking ejects a backend that returns errors fast** — a backend failing every request in milliseconds attracts more traffic under least-connections balancing, not less.
- [ ] **Rate limiting or a connection cap protects the backends** — including from your own retrying clients, which are frequently the largest source of a traffic spike.
- [ ] **Capacity is sized for the loss of an entire zone** — if losing one of three zones puts the remaining two above their breaking point, the redundancy is decorative.
- [ ] **Pre-warming or scaling limits are confirmed for expected traffic spikes** — some managed balancers scale gradually and will shed load during a sudden step change in traffic.

## 8. Observability

- [ ] **Access logs are enabled and their retention and cost are set deliberately** — the balancer's log is often the only record of requests that never reached a backend.
- [ ] **Balancer-generated errors are distinguished from backend errors in dashboards** — a 502 from the balancer and a 500 from the application have entirely different causes and different owners.
- [ ] **Latency is measured as both total time and backend processing time** — the difference between them is queueing and connection time, and a growing gap is the earliest sign of backend saturation.
- [ ] **Per-target metrics are visible, not just aggregates** — one bad backend out of twenty is invisible in the average and obvious per target.
- [ ] **Healthy target count is alerted on with a floor above zero** — alert when it drops below the number needed to serve peak load, not when it reaches zero, by which point you are already down.
- [ ] **Rejected, spilled-over, and surge-queue metrics are monitored** — these count requests the balancer refused before reaching a backend, and they are absent from application metrics entirely.
- [ ] **Trace context is propagated through the balancer** — a request ID assigned at the edge and passed downstream is what lets you follow one user's failing request through every hop.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Topology and listeners | | | Pass / Pass with actions / Fail |
| Health checks | | | Pass / Pass with actions / Fail |
| Timeouts | | | Pass / Pass with actions / Fail |
| Connection draining and deployments | | | Pass / Pass with actions / Fail |
| Session affinity | | | Pass / Pass with actions / Fail |
| TLS termination and re-encryption | | | Pass / Pass with actions / Fail |
| Traffic management and resilience | | | Pass / Pass with actions / Fail |
| Observability | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with an owner before the load balancer is placed in the production traffic path.

## Related checklists

- [TLS Certificate Management](/docs/networking/tls-certificate/)
- [Network Change](/docs/networking/network-change/)
- [Production Readiness Review](/docs/devops/production-readiness/)
- [Kubernetes Deployment](/docs/devops/kubernetes-deployment/)
- [Observability](/docs/operations/observability/)

## References

- [AWS Elastic Load Balancing Documentation](https://docs.aws.amazon.com/elasticloadbalancing/)
- [Google Cloud Load Balancing Documentation](https://cloud.google.com/load-balancing/docs)
- [Envoy Proxy Documentation](https://www.envoyproxy.io/docs/envoy/latest/)
- [NGINX HTTP Upstream Module Reference](https://nginx.org/en/docs/http/ngx_http_upstream_module.html)
- [Google SRE Book — Load Balancing in the Datacenter](https://sre.google/sre-book/load-balancing-datacenter/)
