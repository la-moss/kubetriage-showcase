# KubeTriage Architecture

## Overview

KubeTriage diagnoses Kubernetes incidents from **frozen replay evidence** — pre-recorded outputs from a strict six-tool allowlist. The engine is fully deterministic: no LLM calls, no randomness, no live cluster access during diagnosis, and no write operations.

Deterministic diagnosis is the authority. Provenance and policy enforcement sit on top of that authority. **A6A** adds a versioned admission constitution; **A6B** (through published S2) adds a lifecycle runtime for request preparation, ledger, authority binding, and idempotency. Offline OpenClaw and LLM packages are scaffolds only. No provider boundary is admitted. Real-agent admission remains blocked even though A4A, A4B, A5A, A5B, A6A, and A6B-S2 have passed independent review.

![KubeTriage end-to-end architecture](../images/kubetriage-diagram.png)

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

**Latest reviewed baseline:** `v2.2.0-a6b-s2-reviewed-baseline` (A6B-S2). S3 (capacity and serial execution) is draft only and blocked pending independent review.

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
│  A6 lifecycle runtime (A6B through S2 — in progress)       │
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
- **Live Kubernetes is optional and local-only.** A dedicated kind lab can capture read-only evidence into generated sessions. Capture is not diagnosis. Generated sessions are unreviewed until promoted.
- **OpenClaw is a contract / consumer scaffold.** It is not the currently admitted real-agent path. The LLM adapter is offline-only and does not call a provider.
- **A2 envelope vs A4B v3 path.** `agent_safe_response/v1` is an earlier authority-envelope milestone. Provenance-aware `agent_explanation/v3` under `agent_output_policy/v3` is the later admitted explanation path for provenance-bound rendering.
- **Drift is bounded.** At most one drift recheck per session, always full recomputation. Post-drift confidence must not exceed pre-drift confidence.
- **A5 is complete as an evaluation programme.** Adversarial and metamorphic suites passed independent review. That does not admit a real agent.
- **A6 is in progress.** A6A (admission constitution) is complete. A6B-S2 (lifecycle ledger and idempotency) is the latest published reviewed baseline (`v2.2.0-a6b-s2-reviewed-baseline`). A6B-S3+ and A6C–A6G remain blocked.

## Evaluation corpora

| Corpus | Sessions | Purpose |
| --- | --- | --- |
| Golden | 20 | Development regression; CI replay |
| Holdout | 24 | Evaluation-only; never tuned against |
| Generated | — | Unreviewed live-capture fixtures; not evaluation material until promoted |

Session types include crashloop, imagepull, oom, pending, service_unreachable, probe_failure, config_dependency_failure, insufficient evidence, refusal, tool failure, and drift.

## What is not in this showcase

This repository documents architecture, safety, evaluation summaries, and sample outputs only. It does not include the implementation source, test suite, calibration artifacts, private adversarial fixtures, or governed replay corpora. Those remain in the private implementation repository.
