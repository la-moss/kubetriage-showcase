# Live Fixture Pipeline

This document describes the optional pipeline for generating replay fixtures from a local kind cluster. The pipeline is local-only and not required for KubeTriage diagnosis, evaluation, or the sample outputs in this showcase. Capture produces governed evidence for the diagnostic engine; it is not an observability or monitoring product. This lab workflow is **not** the independently sourced real-incident corpus: real-incident acquisition is a separate support-blind programme and is not model training.

Normal KubeTriage tests, demos, replay, and evaluation remain fully offline and do not require kind, kubectl, Docker, a live cluster, or network access.

## Pipeline overview

```
1. Create dedicated kind cluster (name: kind-kubetriage)
      ↓
2. Deploy controlled broken workload
      ↓
3. Capture read-only evidence
      ↓
4. Produce generated replay session
      ↓
5. Review and redact generated session
      ↓
6. Optionally promote to golden through corpus governance
```

## What the lab provides

The local kind lab creates a dedicated `kind-kubetriage` cluster and deploys controlled broken workloads:

| Scenario | Expected symptom |
| --- | --- |
| ImagePullBackOff | Invalid container image reference |
| CrashLoopBackOff | Container exits with non-zero status |
| Service selector mismatch | Service has no reachable endpoints |

All workloads run in a dedicated lab namespace.

## Read-only evidence capture

Capture maps kubectl output to the six existing logical evidence tools and writes unreviewed generated replay sessions. Capture does **not** diagnose, remediate, or mutate cluster state beyond the initial controlled lab setup.

**Capture is not diagnosis.** Generated sessions are unreviewed and are **not** automatically golden or holdout material. **Replay evidence remains the source of truth** for KubeTriage evaluation.

### Typical workflow

1. Create the dedicated kind cluster.
2. Deploy a named broken-workload scenario.
3. Identify the failing pod in the lab namespace.
4. Run read-only evidence capture into a generated session directory.
5. Run offline diagnosis against the frozen evidence (no live cluster needed).
6. Destroy the cluster when done.

## Governance rules

- **Generated fixtures are unreviewed until reviewed.** A generated session must go through the session review process before it can be admitted to golden or holdout.
- **Generated fixtures must not be added to golden or holdout automatically.** Automatic promotion bypasses the review gate and risks contaminating the evaluation corpus.
- **Live Kubernetes is not model training.** The engine does not learn from live cluster state. Live evidence is converted into frozen replay sessions.
- **Replay evidence remains source of truth.** KubeTriage's correctness is evaluated against deterministic replay, not live cluster behaviour.
- **No remediation is executed by KubeTriage.** The lab deploys broken workloads; KubeTriage reads evidence. No write operations are performed by the engine.
- **Lab setup uses controlled manifests only.** Scenario deployment targets fixed lab manifests — this is not KubeTriage diagnosis or remediation.
- **Capture uses read-only kubectl only.** Mutation verbs are refused. Output is written only to generated session directories (never directly to golden or holdout).

## Relationship to the showcase

This showcase repository includes sample engine outputs from replay sessions (see `sample-output/`). It does not include the lab scripts, capture tooling, or generated session data. Those remain in the private implementation repository.

For the core demo, use the pre-recorded sample outputs. The live lab is an optional extension for audiences who want to see the full capture-to-diagnosis pipeline on a local machine.
