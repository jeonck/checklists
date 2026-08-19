---
title: "Kubernetes Deployment"
description: "Verify a workload manifest is safe, resilient, and operable before it is applied to a production cluster."
icon: "hub"
weight: 240
toc: true
tags: ["kubernetes", "orchestration", "reliability", "devops"]
---

Kubernetes will happily run a badly configured workload for months and then fail it all at once, during a node upgrade or a noisy-neighbour spike. Almost every "Kubernetes outage" is really a missing probe, an absent resource request, or a disruption budget nobody set. Work through this before a workload goes to a production cluster, and again after any significant change to its manifests.

{{< alert context="info" text="**Who runs this:** the owning team, with a platform engineer reviewing cluster-level settings. **When:** before the first production apply, and whenever resources, probes, or scaling behaviour change." />}}

## 1. Workload definition

- [ ] **The correct workload kind is used** — `Deployment` for stateless replicas, `StatefulSet` where stable identity and storage matter, `DaemonSet` for per-node agents, `Job`/`CronJob` for finite work.
- [ ] **Images are referenced by digest and `imagePullPolicy` is deliberate** — a mutable tag with `IfNotPresent` means different nodes can run different code indefinitely.
- [ ] **Replica count is at least two for anything that serves traffic** — a single replica means every node drain is an outage.
- [ ] **Labels follow a consistent scheme including app, component, version, and owner** — selectors, dashboards, network policies, and cost allocation all depend on them.
- [ ] **The pod template has no mutable configuration embedded in it** — configuration comes from `ConfigMap` or `Secret` references, so a config change is reviewable on its own.
- [ ] **A change to a `ConfigMap` triggers a rollout** — either via a checksum annotation on the pod template or immutable, versioned config objects, otherwise pods keep the old values until something unrelated restarts them.
- [ ] **The manifest is version controlled and applied by automation** — nothing reaches production through `kubectl edit` on someone's laptop.

## 2. Resource requests and limits

- [ ] **Every container sets CPU and memory requests based on measured usage** — requests drive scheduling, and an unset request means the scheduler is guessing.
- [ ] **Memory limits are set and equal to or close to the request for latency-sensitive workloads** — memory is incompressible, so exceeding a limit means an immediate `OOMKilled`, not throttling.
- [ ] **CPU limits are applied only where you intend throttling** — an aggressive CPU limit causes latency spikes through CFS throttling even while the node has idle capacity.
- [ ] **The resulting quality-of-service class is understood** — `BestEffort` pods are evicted first under node pressure, which is rarely what you want for a production service.
- [ ] **Resource values have been revisited against real production usage** — copied-from-another-service values are the most common source of both waste and eviction.
- [ ] **Namespace `ResourceQuota` and `LimitRange` allow the workload's peak, including during a rollout** — a rollout temporarily needs headroom for surge pods.
- [ ] **Requests are reconciled against the cluster's node sizes** — a pod requesting more than any node can offer stays `Pending` forever with an unhelpful message.

## 3. Health probes and lifecycle

- [ ] **Readiness probes reflect the ability to serve a real request** — not a static `200` from a handler that ignores dependency state.
- [ ] **Liveness probes do not check downstream dependencies** — a shared dependency outage otherwise restart-loops every healthy pod in the fleet and turns a partial failure into a total one.
- [ ] **A startup probe covers slow initialisation** — otherwise the liveness probe kills a container that was simply still warming up.
- [ ] **Probe timeouts, periods, and failure thresholds are tuned to real timings** — the defaults are frequently too aggressive for JVM and heavy framework start-ups.
- [ ] **`terminationGracePeriodSeconds` exceeds the longest in-flight request plus drain time** — otherwise every deploy severs live connections.
- [ ] **A `preStop` hook gives the service mesh or load balancer time to deregister** — endpoint removal and container termination are concurrent, so a short sleep prevents requests being routed to a shutting-down pod.
- [ ] **The application handles `SIGTERM` and stops accepting new work while draining** — verify by deleting a pod under load and watching for errors.

## 4. Scheduling and availability

- [ ] **A `PodDisruptionBudget` protects the workload during node drains** — without one, a cluster upgrade can evict every replica simultaneously.
- [ ] **The disruption budget cannot deadlock the cluster** — `minAvailable` equal to the replica count blocks all voluntary evictions and stalls node maintenance.
- [ ] **Pod anti-affinity or topology spread constraints distribute replicas across nodes and zones** — two replicas on the same node is one node failure away from an outage.
- [ ] **Tolerations and node selectors are minimal and documented** — a broad toleration can land a workload on a node pool it was never meant to touch.
- [ ] **Priority classes are set deliberately for critical workloads** — so that under node pressure the right things are evicted.
- [ ] **Behaviour during a cluster autoscaler scale-down has been considered** — pods with local storage or restrictive budgets can block node removal indefinitely.

{{< alert context="warning" text="**Blocking:** a production workload with no readiness probe and no PodDisruptionBudget will drop traffic during every routine node upgrade. Neither is optional for a service that carries user requests." />}}

## 5. Configuration and secrets

- [ ] **Secrets come from a secret manager through an operator or CSI driver, or are encrypted in git via a sealed-secrets style tool** — plain base64 in a manifest is encoding, not encryption.
- [ ] **Etcd encryption at rest is enabled on the cluster for secret resources** — confirm with the platform team rather than assuming.
- [ ] **Secrets are mounted as files rather than environment variables where practical** — environment variables leak through crash dumps, child processes, and debug endpoints.
- [ ] **Secret rotation does not require a manual redeploy** — either the application reloads mounted files or rotation triggers a controlled rollout.
- [ ] **Config differences between environments live in an overlay or values file, not in duplicated manifests** — divergence between copies is how staging stops predicting production.
- [ ] **No service account token is mounted unless the pod calls the Kubernetes API** — set `automountServiceAccountToken: false` by default.

## 6. Pod and container security

- [ ] **The pod security context sets `runAsNonRoot`, a numeric `runAsUser`, and `fsGroup` where volumes are written** — and the image actually supports it.
- [ ] **`allowPrivilegeEscalation` is false and all Linux capabilities are dropped, adding back only what is needed** — most workloads need none.
- [ ] **The root filesystem is read-only with explicit `emptyDir` mounts for writable paths** — this removes the simplest persistence mechanism for an attacker.
- [ ] **A seccomp profile is applied** — `RuntimeDefault` at minimum, which is not the default in older clusters.
- [ ] **No host namespaces, host paths, or privileged containers are used** — each of these is effectively a route to the node, and any exception needs written approval.
- [ ] **The namespace enforces Pod Security Admission at the restricted level where possible** — a policy that only warns will be ignored.
- [ ] **The pod's service account has a least-privilege `Role` binding** — never `cluster-admin`, and never a wildcard verb on a wildcard resource.

## 7. Networking and traffic

- [ ] **A default-deny `NetworkPolicy` exists in the namespace, with explicit allow rules** — without a default deny, every pod can reach every other pod in the cluster.
- [ ] **Egress is restricted to the dependencies the service actually needs** — including DNS, which is the rule people forget and then spend an hour debugging.
- [ ] **The `Service` selector matches the pod labels and resolves to the expected endpoints** — verify with `kubectl get endpoints`, since a typo produces a silent black hole.
- [ ] **Ingress terminates TLS with a certificate that renews automatically** — and the renewal has been observed to work at least once.
- [ ] **Timeouts and retry behaviour at the ingress or mesh layer are aligned with the application's own timeouts** — mismatched layers turn one slow request into an amplified retry storm.
- [ ] **Session requirements are explicit** — if the application needs sticky sessions or long-lived connections, the load balancing configuration reflects that.

## 8. Storage and state

- [ ] **`PersistentVolumeClaim` storage classes are chosen deliberately, including the reclaim policy** — a `Delete` policy on production data is one `kubectl delete` from permanent loss.
- [ ] **Access modes match the topology** — a `ReadWriteOnce` volume ties the pod to a single node and constrains rolling updates.
- [ ] **Volume expansion is supported and the procedure has been tested** — a full volume at 2am is not the moment to discover the storage class forbids resizing.
- [ ] **Backups of persistent volumes are configured and a restore has been performed** — snapshots that have never been restored are untested.
- [ ] **`StatefulSet` scale-down behaviour is understood** — orphaned volumes remain and cost money, or are reclaimed and destroy data, depending on configuration.
- [ ] **`emptyDir` usage is bounded with a size limit** — an unbounded `emptyDir` can fill the node disk and evict unrelated pods.

## 9. Rollout, observability, and operations

- [ ] **The rolling update strategy sets `maxUnavailable` and `maxSurge` deliberately** — and the cluster has capacity for the surge.
- [ ] **A failing rollout stops rather than replacing every healthy pod** — `progressDeadlineSeconds` is set and the rollout status is checked by the pipeline.
- [ ] **Rollback to the previous revision has been rehearsed** — and the revision history limit retains enough revisions to do it.
- [ ] **Metrics, logs, and traces from the pod reach the platform's collectors** — confirmed by looking at a dashboard, not by reading configuration.
- [ ] **Horizontal autoscaling targets a metric that actually reflects load** — CPU is a poor proxy for an IO-bound service, and scaling on the wrong signal amplifies incidents.
- [ ] **Autoscaler minimum and maximum replicas are set, and the maximum is affordable and schedulable.**
- [ ] **Alerts exist for crash-looping pods, pending pods, and failed rollouts** — these are the platform-level symptoms that precede user-visible failure.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Workload definition | | | Pass / Pass with actions / Fail |
| Resource requests and limits | | | Pass / Pass with actions / Fail |
| Health probes and lifecycle | | | Pass / Pass with actions / Fail |
| Scheduling and availability | | | Pass / Pass with actions / Fail |
| Configuration and secrets | | | Pass / Pass with actions / Fail |
| Pod and container security | | | Pass / Pass with actions / Fail |
| Networking and traffic | | | Pass / Pass with actions / Fail |
| Storage and state | | | Pass / Pass with actions / Fail |
| Rollout, observability, and operations | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with a named owner before the workload is approved for production traffic.

## Related checklists

- [Production Readiness Review](/docs/devops/production-readiness/)
- [Container Image Hardening](/docs/devops/container-image/)
- [Observability](/docs/operations/observability/)
- [Capacity Planning](/docs/operations/capacity-planning/)
- [Cloud Security](/docs/security/cloud-security/)

## References

- [Kubernetes — Configuration Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)
- [Kubernetes — Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [Kubernetes — Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Kubernetes — Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [Kubernetes — Security Checklist](https://kubernetes.io/docs/concepts/security/security-checklist/)
