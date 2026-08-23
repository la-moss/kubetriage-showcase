# KubeTriage Demo Walkthrough

This guide covers how to demonstrate KubeTriage in 3–10 minutes using the sample outputs in this showcase repository. The full implementation repository runs the same flows offline against frozen replay evidence — no live Kubernetes cluster is required for diagnosis or evaluation.

---

## Three core demo paths

Each path demonstrates a different safety or diagnostic property. Representative output is included under `sample-output/`.

### 1. Normal diagnosis — imagepull incident

**What to show:** [sample-output/engine-imagepull-demo.txt](../sample-output/engine-imagepull-demo.txt)

KubeTriage classifies a frozen evidence bundle as `imagepull` with high confidence (0.88), ranks hypotheses, and records the next read-only diagnostic observation. Same evidence always produces the same report.

**Talking points:**
- Deterministic — run twice, output is identical
- Read-only — no write operations, no remediation
- Six-tool allowlist preserved

### 2. Drift recomputation — class flip

**What to show:** [sample-output/engine-drift-demo.txt](../sample-output/engine-drift-demo.txt)

The hero scenario. Pre-drift evidence supports `crashloop` (confidence 0.88). Post-drift evidence adds image pull failure signals. KubeTriage recomputes from the full merged evidence and the top class changes to `imagepull` (confidence 0.76).

**Talking points:**
- Full recomputation — never patches the previous answer
- Confidence does not inflate after drift (0.88 → 0.76)
- At most one bounded drift recheck per session
- No remediation is executed

### 3. Safety refusal — unsafe cluster context

**What to show:** [sample-output/engine-refusal-demo.txt](../sample-output/engine-refusal-demo.txt)

KubeTriage refuses a request targeting a non-safe cluster context. Refusal is terminal — no diagnostic loop continues, no remediation is attempted.

**Talking points:**
- Safety enforced in code, not prompts
- Refusal is successful safety behaviour
- Context guard requires the dedicated lab context (`kind-kubetriage`) unless an explicit unsafe override is provided

---

## Provenance and evaluation samples (after A4 / A5)

### Provenance-aware accepted explanation (A4B v3)

**What to show:** [sample-output/provenance-aware-explanation-demo.json](../sample-output/provenance-aware-explanation-demo.json)

A real accepted `agent_explanation/v3` admission under `agent_output_policy/v3` for `golden-imagepull-backoff`, with sealed outcome, constrained claims, stable fact IDs, contribution and support-set references, provenance bundle binding, and renderer digest.

### A5A / A5B summaries

- [adversarial-evaluation-summary.json](../sample-output/adversarial-evaluation-summary.json) — 102/102, frozen digest, zero unexpected admissions
- [metamorphic-evaluation-summary.json](../sample-output/metamorphic-evaluation-summary.json) — 67/67 suite v2, pure vs mixed removal distinction
- [confidence-decomposition-demo.json](../sample-output/confidence-decomposition-demo.json) — reviewed mixed-information removal (0.72 → 0.88 via dampening clear)

---

## Earlier / legacy layers (clearly labelled)

### A2 agent-safe response envelope

**What to show:** [sample-output/agent-safe-response-demo.json](../sample-output/agent-safe-response-demo.json)

Closed `agent_safe_response/v1` authority envelope. Valid earlier milestone. Local IDs such as `ev-001` are envelope-local and are **not** current A4B stable fact identity. A4B v3 provenance enforcement supersedes this for the admitted provenance-aware explanation path.

### A3C constrained consumer (offline)

**What to show:** [sample-output/agent-consumer-demo.txt](../sample-output/agent-consumer-demo.txt)

Offline constrained explanation under `agent_output_policy/v2`. Not the v3 provenance-aware path. Not real-agent admission.

### Legacy LLM policy scaffold

**What to show:** [sample-output/llm-policy-demo.txt](../sample-output/llm-policy-demo.txt)

Offline prompt/policy scaffold. **No real LLM is called.** Not the admitted v3 path. Not evidence of real-agent readiness.

---

## What to explain while demoing

### Core properties

- **Deterministic.** The same evidence always produces the same diagnosis, confidence, hypothesis ranking, and report. No randomness, no wall-clock dependence, no hidden state.

- **Read-only.** KubeTriage never executes write operations. All write and mutation verbs (`apply`, `patch`, `delete`, `exec`, `scale`, `rollout restart`, etc.) are blocked in code. The engine produces reports and refusals, never cluster changes.

- **Replay-first.** Every test and evaluation runs against frozen evidence files. No live cluster is needed for the core demo.

- **A5 complete; A6 paused; admission blocked.** Adversarial and metamorphic evaluation passed independent review. A6A (constitution) and A6B-S2 (lifecycle ledger) are published reviewed A6 baselines. A6B-S3+ and A6C–A6G remain blocked. Operational and human gates still block real-agent admission. Current validation work is support-blind real-incident corpus acquisition, not A6 completion.

### Safety model

See [safety-model.md](safety-model.md) for the full safety model. Key points:

- Six-tool allowlist: `events_tail`, `describe_pod`, `describe_deploy`, `logs`, `get_yaml`, `top_pod`
- Seven incident classes: `crashloop`, `imagepull`, `pending`, `service_unreachable`, `oom`, `probe_failure`, `config_dependency_failure`
- Refusal is terminal — no diagnostic loop continues after refusal
- Golden and holdout are governed corpora; generated sessions are unreviewed until promoted

### Evaluation model

- **Golden corpus** (20 sessions) is used for development regression.
- **Holdout corpus** (24 sessions) is evaluation-only. Engine behaviour must not be tuned against holdout results.
- Latest replay state is golden 20/20 and holdout 24/24, with zero false-high-confidence results in calibration.
- **Calibration** evaluates base (non-drift) and drift metrics separately — never mixed as a single headline metric.

---

## Hero scenario: drift class flip

This is the strongest single-session demo. It shows diagnosis, drift, safety, and determinism in one pass.

1. Pre-drift evidence contains strong crashloop signals. KubeTriage classifies as `crashloop` with confidence 0.88.
2. Post-drift evidence adds image pull failure signals. Crashloop signals remain but imagepull signals are now dominant.
3. KubeTriage recomputes from the full merged evidence — fact extraction, classification, hypothesis ranking, confidence scoring.
4. The top class changes deterministically from `crashloop` to `imagepull`. The `changed` flag is `true`.
5. Confidence does not inflate. Post-drift confidence (0.76) is lower than pre-drift (0.88) because merged evidence contains contradictions.
6. No remediation is executed. A human decides what to do next.

No autonomous agent is currently connected to or admitted by KubeTriage. A4A/A4B/A5A/A5B have passed independent review and A5 is complete as an evaluation programme. A6A (admission constitution) and A6B-S2 (lifecycle ledger and idempotency) are published reviewed A6 lifecycle baselines (`v2.2.0-a6b-s2-reviewed-baseline`). A6B-S3+ and A6C–A6G remain blocked. Real-agent admission remains blocked by operational isolation review and human-approval gates. **Current diagnostic baseline:** `v2.8.0-a6b-pre-s4-result-inventory-reviewed-baseline`. Support-blind real-incident corpus acquisition is underway. The V2 analysis plan is frozen; V2 diagnostic execution has not begun.

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

> KubeTriage is a deterministic evidence engine for bounded incident diagnosis. Today it is validated on controlled Kubernetes replay evidence, with support-blind real-incident corpus acquisition underway ahead of blind real-world evaluation; it is not an observability platform, not a remediator, and not an AI that invents a diagnosis.

---

## What KubeTriage does not do (demo talking points)

| Claim | Reality |
| --- | --- |
| "It calls an LLM" | No. Engine logic is deterministic code. Zero LLM imports in the engine. |
| "It runs kubectl" | Not during diagnosis. Replay backend only. Live capture is a separate local-lab pipeline. |
| "It fixes things" | No. Strictly read-only. All write verbs are refused. |
| "It trains on data" | No. Evidence creates replay fixtures (after human review), not model updates. |
| "It adapts automatically" | No. Engine changes only through deliberate, reviewed code changes. |
| "A5 means a real agent is ready" | No. A5 is complete as evaluation; A6 lifecycle work is paused; real-agent admission remains blocked. |
| "A6B-S2 means a provider is connected" | No. S2 is process-local ledger and idempotency only — no provider SDK or network calls. |
| "It replaces observability" | No. It consumes governed evidence and produces diagnosis; it does not collect telemetry. |
| "It is production validated / real-world proven" | No. Controlled replay evaluation is implemented; real-incident acquisition is in progress; V2 blind evaluation has not begun. |
| "It already diagnoses networking / identity / multi-cloud" | No. Broader evidence domains are architectural scope; only Kubernetes-native replay is implemented. |
