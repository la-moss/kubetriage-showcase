# KubeClaw Demo Walkthrough

This guide covers how to demonstrate KubeClaw in 3–10 minutes using the sample outputs in this showcase repository. The full implementation repository runs the same flows offline against frozen replay evidence — no live Kubernetes cluster is required for diagnosis or evaluation.

---

## Three core demo paths

Each path demonstrates a different safety or diagnostic property. Representative output is included under `sample-output/`.

### 1. Normal diagnosis — imagepull incident

**What to show:** [sample-output/engine-imagepull-demo.txt](../sample-output/engine-imagepull-demo.txt)

KubeClaw classifies a frozen evidence bundle as `imagepull` with high confidence (0.88), ranks hypotheses, and suggests the next read-only diagnostic step. Same evidence always produces the same report.

**Talking points:**
- Deterministic — run twice, output is identical
- Read-only — no write operations, no remediation
- Six-tool allowlist preserved

### 2. Drift recomputation — class flip

**What to show:** [sample-output/engine-drift-demo.txt](../sample-output/engine-drift-demo.txt)

The hero scenario. Pre-drift evidence supports `crashloop` (confidence 0.88). Post-drift evidence adds image pull failure signals. KubeClaw recomputes from the full merged evidence and the top class changes to `imagepull` (confidence 0.76).

**Talking points:**
- Full recomputation — never patches the previous answer
- Confidence does not inflate after drift (0.88 → 0.76)
- At most one bounded drift recheck per session
- No remediation is executed

### 3. Safety refusal — unsafe cluster context

**What to show:** [sample-output/engine-refusal-demo.txt](../sample-output/engine-refusal-demo.txt)

KubeClaw refuses a request targeting a non-safe cluster context. Refusal is terminal — no diagnostic loop continues, no remediation is attempted.

**Talking points:**
- Safety enforced in code, not prompts
- Refusal is successful safety behaviour
- Context guard requires the dedicated lab context unless an explicit unsafe override is provided

---

## Agent and LLM layers

### OpenClaw agent consumer

**What to show:** [sample-output/agent-consumer-demo.txt](../sample-output/agent-consumer-demo.txt)

A deterministic prototype that reads KubeClaw output and produces a bounded
human-readable explanation. Every built-in explanation includes a safety
boundary attestation:

- I did not call kubectl
- I did not modify the cluster
- I did not override KubeClaw's diagnosis
- I did not invent evidence
- I did not execute remediation

KubeClaw remains the diagnostic authority. The target boundary permits
explanation but prohibits override, invention, mutation, and remediation. The
current scaffold does not fully enforce that boundary and is not sufficient for
real-agent admission.

Deterministic OpenClaw output can also be mapped into the strict
`agent_safe_response/v1` envelope and validated against the closed contract
before being exposed. The envelope carries exactly one outcome (`diagnosis`,
`insufficient_evidence`, or `refusal`), required drift/evidence/safety/
provenance sections, and no remediation or command fields. This is OpenClaw
output validation, not real-agent admission — admission remains blocked.

### LLM policy boundary

**What to show:** [sample-output/llm-policy-demo.txt](../sample-output/llm-policy-demo.txt)

An offline scaffold for a future LLM explanation layer. It builds prompts from
KubeClaw results and applies deterministic string/value checks to candidate
explanations. No real LLM is called. The checks reject selected known
remediation patterns, but semantic policy hardening remains required.

---

## What to explain while demoing

### Core properties

- **Deterministic.** The same evidence always produces the same diagnosis, confidence, hypothesis ranking, and report. No randomness, no wall-clock dependence, no hidden state.

- **Read-only.** KubeClaw never executes write operations. All write and mutation verbs (`apply`, `patch`, `delete`, `exec`, `scale`, `rollout restart`, etc.) are blocked in code. The engine produces reports and refusals, never cluster changes.

- **Replay-first.** Every test and evaluation runs against frozen evidence files. No live cluster is needed for the core demo. This makes the test suite fast, reproducible, and CI-friendly.

### Safety model

See [safety-model.md](safety-model.md) for the full safety model. Key points:

- Six-tool allowlist: `events_tail`, `describe_pod`, `describe_deploy`, `logs`, `get_yaml`, `top_pod`
- Seven incident classes: `crashloop`, `imagepull`, `pending`, `service_unreachable`, `oom`, `probe_failure` (readiness/liveness grouped), `config_dependency_failure` (ConfigMap/Secret grouped)
- Refusal is terminal — no diagnostic loop continues after refusal
- Golden and holdout are governed corpora; generated sessions are unreviewed until promoted

### Evaluation model

- **Golden corpus** (20 sessions) is used for development regression.
- **Holdout corpus** (24 sessions) is evaluation-only. Engine behaviour must not be tuned against holdout results.
- **44 governed sessions total**; latest replay state is golden 20/20 and holdout 24/24, with zero false-high-confidence results in calibration.
- **Calibration** evaluates base (non-drift) and drift metrics separately — never mixed as a single headline metric.

---

## Hero scenario: drift class flip

This is the strongest single-session demo. It shows diagnosis, drift, safety, and determinism in one pass.

1. Pre-drift evidence contains strong crashloop signals. KubeClaw classifies as `crashloop` with confidence 0.88.
2. Post-drift evidence adds image pull failure signals. Crashloop signals remain but imagepull signals are now dominant.
3. KubeClaw recomputes from the full merged evidence — fact extraction, classification, hypothesis ranking, confidence scoring.
4. The top class changes deterministically from `crashloop` to `imagepull`. The `changed` flag is `true`.
5. Confidence does not inflate. Post-drift confidence (0.76) is lower than pre-drift (0.88) because merged evidence contains contradictions.
6. No remediation is executed. A human (or a future evaluated agent) decides what to do next.

No real autonomous AI agent currently exists. The strict
`agent_safe_response/v1` envelope and OpenClaw output validation are in place,
but real-agent admission is blocked until policy hardening (A3), stable
citation/provenance hardening (A4), adversarial evaluation (A5),
operational-isolation review, and human-approval gates are complete.

---

## Optional: live fixture lab

For audiences who want to see evidence capture from a real (local) cluster, the optional kind lab pipeline is described in [live-fixture-pipeline.md](live-fixture-pipeline.md):

```
create local kind cluster → deploy broken workload → capture read-only evidence
  → generated replay session → offline diagnosis → destroy cluster
```

Live capture is local-only, optional, and not required for the core demo. Replay evidence remains the source of truth.

---

## Interview pitch

> KubeClaw is not an AI that randomly runs kubectl. It is the deterministic, evaluated, safety-bounded diagnostic layer that an AI would need before it should be trusted near Kubernetes.

---

## What KubeClaw does not do (demo talking points)

| Claim | Reality |
| --- | --- |
| "It calls an LLM" | No. Engine logic is deterministic code. Zero LLM imports in the engine. |
| "It runs kubectl" | Not during diagnosis. Replay backend only. Live capture is a separate local-lab pipeline. |
| "It fixes things" | No. Strictly read-only. All write verbs are refused. |
| "It trains on data" | No. Evidence creates replay fixtures (after human review), not model updates. |
| "It adapts automatically" | No. Engine changes only through deliberate, reviewed code changes. |
