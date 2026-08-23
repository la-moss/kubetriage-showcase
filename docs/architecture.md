# KubeTriage Architecture

## Overview

KubeTriage is a **deterministic diagnostic engine for cloud-native systems**. It diagnoses from **frozen, read-only evidence** — not from live cluster access during diagnosis. The engine has no LLM calls, no randomness, and no write operations.

**Current implementation:** governed inputs are Kubernetes-native replay evidence — pre-recorded outputs from a strict six-tool allowlist — and seven bounded incident classes.

**Architectural scope:** the diagnostic pipeline is designed to accommodate additional governed operational evidence sources over time while preserving determinism, provenance, replayability, and bounded authority. Those additional sources are **not implemented**.

Deterministic diagnosis is the authority. Provenance and policy enforcement sit on top of that authority. **Current diagnostic baseline:** `v2.8.0-a6b-pre-s4-result-inventory-reviewed-baseline`. **A6A** adds a versioned admission constitution; **A6B** (through published S2) adds a lifecycle runtime for request preparation, ledger, authority binding, and idempotency. Agent-admission work is paused while support-blind real-incident corpus acquisition proceeds. Offline OpenClaw and LLM packages are scaffolds only. No provider boundary is admitted. Real-agent admission remains blocked even though A4A, A4B, A5A, A5B, A6A, and A6B-S2 have passed independent review.

## Where KubeTriage fits

KubeTriage sits in the diagnostic reasoning layer. Observability and infrastructure systems produce operational evidence; KubeTriage consumes governed subsets of that evidence and produces a replayable diagnosis. It does not replace observability, monitoring, or telemetry collection, and it does not remediate.

```text
operational evidence
  → validation
  → fact extraction
  → hypothesis evaluation
  → confidence
  → provenance
  → replayable diagnosis
```

Potential evidence domains for that reasoning (examples only — not a claim of current coverage) include workload/resource state, configuration, Kubernetes events, logs, service and endpoint state, networking evidence, DNS, identity, policy, deployment history, and infrastructure state. Today only the Kubernetes-native six-tool replay path is implemented.

| Responsibility | In scope for KubeTriage |
| --- | --- |
| Validate and redact governed evidence | Yes |
| Extract typed facts with exact citations | Yes |
| Rank hypotheses and calculate confidence | Yes |
| Produce replayable, provenance-bound diagnostic results | Yes |
| Collect telemetry / monitor infrastructure | No |
| Remediate or mutate cluster state | No |

Diagnosis and explanation remain separate: the engine decides the diagnostic result; any future explanation consumer may describe an admitted result but must not change it.

## Current admitted chain

```text
frozen read-only evidence
  → canonical redacted evidence artifacts
  → typed facts with exact citations
  → deterministic classification
  → hypothesis ranking
  → confidence decomposition
  → immutable run manifest
  → claim / fact / evidence provenance
  → contribution replay
  → identity-bound support sets
  → provenance-aware policy v3
  → immutable validated capability
  → deterministic renderer
```

## A6 lifecycle layer (A6B through S2)

After the provenance-aware v3 path, the **A6 controlled explanation-agent admission** programme adds lifecycle controls before any provider could connect:

```text
admission request
  → A6B-S1: canonical preparation and verification
  → A6B-S2: register, ledger, authority binding, idempotent replay
  → (A6B-S3+ blocked) capacity slot, serial execution, A6A invocation
  → A6A: constitutional admission (permissions, prohibitions, admitted claims)
  → deterministic renderer (only if constitutionally admitted)
```

**Latest reviewed A6 lifecycle baseline:** `v2.2.0-a6b-s2-reviewed-baseline` (A6B-S2). S3 (capacity and serial execution) is draft only and blocked pending independent review. A6 work is paused; it is not the current validation programme. **Current diagnostic baseline:** `v2.8.0-a6b-pre-s4-result-inventory-reviewed-baseline`.

## Layer separation

```text
┌────────────────────────────────────────────────────────────┐
│  Deterministic engine (diagnostic authority)               │
│  safety / validation / redaction → state machine → report  │
│  no LLM · no remediation · replay-first                    │
└────────────────────────────┬───────────────────────────────┘
                             │
                             ▼
┌────────────────────────────────────────────────────────────┐
│  Provenance / admission enforcement                        │
│  run manifest · fact provenance · contribution replay      │
│  support-set identity · agent_output_policy/v3             │
│  capability mint · deterministic renderer                  │
└────────────────────────────┬───────────────────────────────┘
                             │
                             ▼
┌────────────────────────────────────────────────────────────┐
│  A6 lifecycle runtime (A6B through S2 — paused)            │
│  request preparation · ledger · idempotency · transitions  │
│  A6A constitution as sole semantic admission authority     │
│  no provider · no remediation · real-agent still blocked   │
└────────────────────────────┬───────────────────────────────┘
                             │
                             ▼
┌────────────────────────────────────────────────────────────┐
│  Offline OpenClaw / LLM scaffolds (not admitted agent)     │
│  agent_safe_response/v1 mapping · constrained v2 demo      │
│  offline LLM adapter string/value checks — no provider     │
└────────────────────────────┬───────────────────────────────┘
                             │
                             ▼
┌────────────────────────────────────────────────────────────┐
│  Future provider boundary — NOT PRESENT                    │
│  real-agent admission blocked by operational/human gates   │
└────────────────────────────────────────────────────────────┘
```

## Deterministic engine data flow

```
Replay Evidence (frozen tool outputs)
        │
        ▼
┌─────────────────────────────────────────────────────┐
│  Safety / Validation / Redaction                     │
│  Context guard, namespace guard                      │
│  Tool spoofing, homoglyph, injection detection       │
│  Secret data, JWT, PEM redaction                     │
│  Six-tool allowlist (asserted)                       │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  Deterministic State Machine                         │
│  SIGNAL_INGESTION → FACT_EXTRACTION → CLASSIFICATION │
│  → HYPOTHESIS_RANKING → CONFIDENCE_SCORING           │
│  → STOP_OR_NEXT                                      │
│       ├── FINAL_REPORT                               │
│       └── DRIFT_RECHECK (at most once) → re-enter    │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  Diagnostic report (JSON + Markdown)                 │
│  or terminal Refusal                                 │
└─────────────────────────────────────────────────────┘
```

## Key invariants

- **Replay evidence is the source of truth.** Diagnosis runs against frozen tool outputs. The engine does not call kubectl during replay.
- **Evidence model is governed and currently Kubernetes-native.** Additional governed operational evidence sources are architectural scope only; they are not present in the current implementation.
- **KubeTriage is not an observability platform.** Capture (when used) is a separate local lab concern; diagnosis consumes frozen evidence and does not replace systems that produce it.
- **Seven incident classes are the current proof point.** They are not the permanent architectural boundary of the diagnostic engine.
- **Live Kubernetes is optional and local-only.** A dedicated kind lab can capture read-only evidence into generated sessions. Capture is not diagnosis. Generated sessions are unreviewed until promoted.
- **OpenClaw is a contract / consumer scaffold.** It is not the currently admitted real-agent path. The LLM adapter is offline-only and does not call a provider.
- **A2 envelope vs A4B v3 path.** `agent_safe_response/v1` is an earlier authority-envelope milestone. Provenance-aware `agent_explanation/v3` under `agent_output_policy/v3` is the later admitted explanation path for provenance-bound rendering.
- **Drift is bounded.** At most one drift recheck per session, always full recomputation. Post-drift confidence must not exceed pre-drift confidence.
- **A5 is complete as an evaluation programme.** Adversarial and metamorphic suites passed independent review. That does not admit a real agent.
- **A6 is paused.** A6A (admission constitution) is complete. A6B-S2 (lifecycle ledger and idempotency) is the latest published reviewed A6 lifecycle baseline (`v2.2.0-a6b-s2-reviewed-baseline`). A6B-S3+ and A6C–A6G remain blocked. Current validation work is support-blind real-incident corpus acquisition, not A6 completion.
- **Current diagnostic baseline is v2.8.** Frozen tag `v2.8.0-a6b-pre-s4-result-inventory-reviewed-baseline`. Real-incident acquisition and admission have not changed or tuned diagnostic behaviour. The V2 analysis plan is frozen; V2 diagnostic execution has not begun.

## Evaluation corpora

Golden, holdout, and generated sessions are **controlled** evaluation / capture
material. They are not the independently sourced real-incident corpus.
Real-incident acquisition is a separate support-blind programme; V2 blind
real-incident evaluation has not begun.

| Corpus | Sessions | Purpose |
| --- | --- | --- |
| Golden | 20 | Development regression; CI replay |
| Holdout | 24 | Evaluation-only; never tuned against |
| Generated | — | Unreviewed live-capture fixtures; not evaluation material until promoted |

Session types include crashloop, imagepull, oom, pending, service_unreachable, probe_failure, config_dependency_failure, insufficient evidence, refusal, tool failure, and drift.

## What is not in this showcase

This repository documents architecture, safety, evaluation summaries, and sample outputs only. It does not include the implementation source, test suite, calibration artifacts, private adversarial fixtures, or governed replay corpora. Those remain in the private implementation repository.
