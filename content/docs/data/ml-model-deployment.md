---
title: "ML Model Deployment"
description: "Verify a trained model is reproducible, safe to serve, monitored for drift, and reversible before it affects users."
icon: "model_training"
weight: 630
toc: true
tags: ["mlops", "machine-learning", "deployment", "monitoring"]
---

A model that scores well offline can still be a bad deployment: the features it sees in production differ from the ones it trained on, nobody can reproduce the artefact six months later, and there is no way back once traffic is on it. This checklist treats the model as a production service with unusual failure modes — silent degradation, feedback loops, and correctness that depends on data you do not control. Work through it before the model serves real decisions.

{{< alert context="info" text="**Who runs this:** the owning ML engineer or data scientist, with a platform or SRE reviewer and, for consequential decisions, a responsible-AI reviewer. **When:** before the first production rollout, and again for any retrain that changes features, architecture, or training data sources." />}}

## 1. Reproducibility and versioning

- [ ] **The exact training dataset is versioned and addressable** — a snapshot, a table version, or a manifest of file hashes, so the run can be repeated rather than approximated.
- [ ] **Training code, configuration, and hyperparameters are committed and linked to the artefact** — a model whose training command lives in someone's notebook history is not reproducible.
- [ ] **The training environment is pinned** — library versions, base image, and hardware type, because a minor version bump in a numerical library will move your metrics.
- [ ] **Random seeds are set and the sources of remaining nondeterminism are documented** — you should be able to say how much run-to-run variance is expected.
- [ ] **Every registered model version records its lineage** — training data version, code commit, evaluation results, and who approved it.
- [ ] **A retrain of the current production model has been demonstrated end to end** — if you cannot rebuild the running model, you cannot fix it under pressure.
- [ ] **Model artefacts are stored immutably with integrity checks** — overwriting a version in place makes every downstream record of which model served a request a lie.

## 2. Features and training/serving skew

- [ ] **Training and serving compute features from the same code path** — a reimplementation of the feature logic in the serving language is the single most common source of skew.
- [ ] **Point-in-time correctness is enforced when building training data** — joining a feature at its current value rather than its value at event time leaks the future and inflates offline metrics.
- [ ] **No feature is derived from information unavailable at prediction time** — audit each feature for label leakage explicitly, including fields updated after the outcome is known.
- [ ] **Feature distributions in training and in live serving have been compared directly** — log a sample of production feature vectors and test them against the training distribution before rollout.
- [ ] **The feature store serves the same values online and offline** — and the freshness lag of each online feature is measured, because a feature that is an hour stale at serving time was never an hour stale in training.
- [ ] **Missing and out-of-range feature values have defined behaviour** — an imputation rule agreed in advance, not whatever the framework does by default when a lookup misses.
- [ ] **Feature transformations are versioned with the model** — a model loaded with the wrong scaler or encoder produces confident nonsense rather than an error.

{{< alert context="danger" text="**Blocking:** if production feature vectors have never been compared against the training distribution, do not release. Training/serving skew produces a model that passes every offline test and quietly performs far worse than measured on real traffic." />}}

## 3. Offline evaluation

- [ ] **The evaluation split reflects how the model will be used** — time-based splits for anything with temporal structure, because a random split on time-series data measures nothing useful.
- [ ] **A held-out test set is used once, at the end** — a test set consulted repeatedly during tuning has become a validation set and no longer estimates generalisation.
- [ ] **The primary metric is tied to the business outcome and its threshold is justified** — a chosen operating point on the precision/recall curve, with the cost of each error type written down.
- [ ] **Performance is reported per segment, not only in aggregate** — by customer tier, region, device, or any group large enough to matter, since aggregate parity can hide a segment that got much worse.
- [ ] **The new model is compared against the current production model and a trivial baseline** — beating a heuristic or the previous version is the bar, not beating random.
- [ ] **Calibration is checked where the score is consumed as a probability** — a ranking model used as a probability threshold will misprice every downstream decision.
- [ ] **Robustness to realistic input degradation is tested** — missing fields, delayed features, and stale lookups, since these will occur in production far more often than in your test set.

## 4. Serving infrastructure

- [ ] **Latency at p95 and p99 is measured under expected concurrency** — mean inference time on an idle machine tells you nothing about behaviour under load.
- [ ] **The model server has request timeouts and a bounded queue** — an unbounded queue converts a slow model into an outage that spreads to every caller.
- [ ] **There is a defined fallback when inference fails or times out** — a cached prediction, a heuristic, or an explicit degraded response, decided by the product owner rather than by an exception handler.
- [ ] **Batch size, concurrency, and hardware are tuned together and the cost per thousand predictions is known** — GPU serving costs are easy to justify and easy to forget.
- [ ] **Model loading happens at start-up and readiness reflects it** — a container that accepts traffic before the weights are in memory will time out its first requests.
- [ ] **Input validation rejects malformed requests before inference** — including payload size limits, since an unbounded input is both a correctness and a denial-of-service problem.
- [ ] **Autoscaling is configured with limits, and the maximum is affordable** — verify the ceiling against the monthly budget, not just the quota.

## 5. Rollout, shadow, and canary

- [ ] **The model runs in shadow mode against live traffic before it serves anyone** — predictions logged and compared, no user impact, for long enough to cover a full weekly cycle.
- [ ] **Shadow results are compared to production on both prediction distribution and latency** — a distribution shift between shadow and current production is a defect to investigate, not a rounding difference.
- [ ] **Rollout is progressive with a defined traffic ramp** — a canary percentage, a soak period at each step, and explicit criteria to advance.
- [ ] **Automated abort criteria are configured before the ramp starts** — error rate, latency, and a leading business metric, each with a threshold that triggers rollback without a meeting.
- [ ] **Assignment to model variants is deterministic per user** — a user flipping between two models mid-session produces incoherent experiences and unusable experiment results.
- [ ] **The online experiment is powered for the metric that matters** — decide the sample size and duration in advance, because stopping when the number looks good guarantees false positives.
- [ ] **Offline and online results are reconciled after the ramp** — a model that improved offline but not online is a lesson about your evaluation setup, and it should be recorded.

## 6. Rollback and model governance

- [ ] **The previous model version remains deployable and has been rolled back to in a rehearsal** — keeping the artefact is not the same as being able to serve it.
- [ ] **Rollback is a single documented action that does not require a retrain** — and it works even when the training pipeline is broken.
- [ ] **Rollback is safe with respect to data written by the new model** — if downstream systems stored scores in a new format, reverting the model is not enough.
- [ ] **Every prediction is logged with the model version that produced it** — without this, no incident involving model output can be scoped after the fact.
- [ ] **A model registry holds the promotion state of each version** — staging, production, archived, with an approval recorded for each transition.
- [ ] **Retraining is triggered by a defined policy** — a schedule, a drift threshold, or a performance floor, and an automatic retrain never promotes itself without passing the same gates.

## 7. Monitoring, drift, and feedback loops

- [ ] **Input feature drift is monitored per feature against the training distribution** — with a statistical test and a threshold, since eyeballing a hundred histograms does not scale.
- [ ] **Prediction distribution is monitored continuously** — it is the earliest available signal, since it needs no labels and moves before outcomes do.
- [ ] **Model quality is measured against delayed ground truth as labels arrive** — and the label delay is documented, because a model can be wrong for weeks before the metric can see it.
- [ ] **Segment-level performance is monitored, not just the aggregate** — a model can hold its overall score while failing badly for one customer group.
- [ ] **Feature pipeline health is alerted on separately from model health** — most production model incidents are actually upstream data incidents.
- [ ] **Feedback loops are identified and controlled** — when the model's own outputs influence its future training data, add exploration or holdout traffic so the training set does not collapse onto past decisions.
- [ ] **A runbook exists for a suspected model incident** — how to check for drift, how to see recent predictions, and how to roll back, written for whoever is on call rather than for its author.

## 8. Responsible AI, security, and compliance

- [ ] **Intended use, out-of-scope use, and known limitations are documented as a model card** — and it is written before launch, not assembled during an audit.
- [ ] **Fairness metrics are evaluated across protected or sensitive groups where the decision affects people** — with the chosen metric justified, since the common definitions cannot all hold at once.
- [ ] **A human review or appeal path exists for consequential automated decisions** — credit, employment, health, and access decisions in particular.
- [ ] **The lawful basis and consent for the training data are confirmed** — including third-party datasets and any personal data used for features.
- [ ] **Personal data is minimised in features and predictions, and retention is enforced by a job** — prediction logs are frequently the largest unmanaged store of personal data in a system.
- [ ] **The model endpoint authenticates and authorises callers, and is rate-limited** — an open scoring endpoint invites both abuse and model extraction.
- [ ] **Adversarial and prompt-injection risks are assessed for models handling untrusted input** — including tests for the outputs that would be most damaging if produced.
- [ ] **Explainability is available at the level the use case requires** — global feature importance for review, and per-prediction explanation where a decision must be justified to the person affected.

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Reproducibility and versioning | | | Pass / Pass with actions / Fail |
| Features and training/serving skew | | | Pass / Pass with actions / Fail |
| Offline evaluation | | | Pass / Pass with actions / Fail |
| Serving infrastructure | | | Pass / Pass with actions / Fail |
| Rollout, shadow, and canary | | | Pass / Pass with actions / Fail |
| Rollback and model governance | | | Pass / Pass with actions / Fail |
| Monitoring, drift, and feedback loops | | | Pass / Pass with actions / Fail |
| Responsible AI, security, and compliance | | | Pass / Pass with actions / Fail |

Record every "Pass with actions" as a dated ticket with a named owner before the model is promoted to full traffic.

## Related checklists

- [Data Pipeline Review](/docs/data/data-pipeline/)
- [Data Quality](/docs/data/data-quality/)
- [Production Readiness Review](/docs/devops/production-readiness/)
- [Observability](/docs/operations/observability/)
- [Cloud Cost Optimization](/docs/cloud/cloud-cost-optimization/)

## References

- [Google Cloud — MLOps: continuous delivery and automation pipelines in machine learning](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning)
- [Google — Rules of Machine Learning](https://developers.google.com/machine-learning/guides/rules-of-ml)
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [MLflow documentation](https://mlflow.org/docs/latest/)
- [Amazon SageMaker AI Developer Guide](https://docs.aws.amazon.com/sagemaker/latest/dg/whatis.html)
