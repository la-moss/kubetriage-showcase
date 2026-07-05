# KubeClaw Architecture

## Overview

KubeClaw diagnoses Kubernetes incidents from **frozen replay evidence** — pre-recorded outputs from a strict six-tool allowlist. The engine is fully deterministic: no LLM calls, no randomness, no live cluster access during diagnosis, and no write operations.

External AI systems may consume KubeClaw results through a validated contract boundary. They can explain reports and suggest read-only investigation steps, but cannot bypass safety guards or override the engine's diagnosis.

## Data flow

```
Replay Evidence (frozen tool outputs)
        │
        ▼
┌─────────────────────────────────────────────────────┐
│  Safety / Validation / Redaction                     │
│                                                      │
│  Context guard, namespace guard                      │
│  Tool spoofing, homoglyph, injection detection       │
│  Secret data, JWT, PEM redaction                     │
│  Six-tool allowlist (asserted)                       │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  Deterministic State Machine                         │
│                                                      │
│  SIGNAL_INGESTION                                    │
│       ↓                                              │
│  FACT_EXTRACTION                                     │
│       ↓                                              │
│  CLASSIFICATION                                      │
│       ↓                                              │
│  HYPOTHESIS_RANKING                                  │
│       ↓                                              │
│  CONFIDENCE_SCORING                                  │
│       ↓                                              │
│  STOP_OR_NEXT                                        │
│       ├── FINAL_REPORT (no drift evidence)           │
│       └── DRIFT_RECHECK → re-enter FACT_EXTRACTION   │
│              (at most once per run)                   │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  Output                                              │
│                                                      │
│  Diagnostic report (JSON + Markdown)                 │
│  Or: Refusal object (terminal, no diagnosis)         │
└─────────────────────────────────────────────────────┘

                  ┌──────────────────┐
                  │  OpenClaw        │
                  │  Contract Layer  │
                  │                  │
                  │  Schema-validated│
                  │  agent boundary  │
                  └──────────────────┘
                         │
              External AI agents call
              KubeClaw through this
              validated boundary.
              They cannot bypass safety.
```

## Optional layers

Beyond the core engine, two prototype layers demonstrate safe AI integration:

```
KubeClaw engine (deterministic diagnosis)
        │
        ▼
OpenClaw agent consumer (deterministic explainer + policy guard)
        │
        ▼
LLM adapter (offline prompt builder + candidate policy validation)
        │
        ▼
[future] LLM provider — explanation only, never diagnosis
```

The agent consumer and LLM adapter are **not** part of the engine. They do not call kubectl, do not remediate, and do not change KubeClaw's diagnosis.

## Key invariants

- **Replay evidence is the source of truth.** All diagnosis runs against frozen tool outputs. The engine does not call kubectl or access live clusters during replay.

- **Live Kubernetes is optional and local-only.** A dedicated kind lab can capture read-only evidence into generated replay sessions. Capture is not diagnosis. Generated sessions are unreviewed until promoted through corpus governance.

- **OpenClaw is a contract boundary.** External AI agents consume KubeClaw through validated JSON schemas. The contract enforces the tool allowlist, blocks write verbs, validates payloads, and rejects unknown tools or fields.

- **Agent evaluation is separate from engine evaluation.** Engine evaluation measures classification accuracy, confidence calibration, and replay correctness. Agent evaluation measures whether an external agent preserves diagnoses, respects refusals, avoids invented facts, and avoids unsafe remediation.

- **Drift is bounded.** At most one drift recheck per session. Drift performs full recomputation from merged evidence — it never patches the previous answer. Post-drift confidence must not exceed pre-drift confidence.

## Evaluation corpora

| Corpus | Purpose |
| --- | --- |
| Golden | Development regression; CI replay |
| Holdout | Evaluation-only; engine behaviour must not be tuned against holdout results |
| Generated | Unreviewed live-capture fixtures; not evaluation material until reviewed and promoted |

Session types covered include crashloop, imagepull, pending, service_unreachable, oom, insufficient evidence, refusal, tool failure, and drift.

## What is not in this showcase

This repository documents architecture, safety, and sample outputs only. It does not include the implementation source, test suite, calibration artifacts, or governed replay corpora. Those remain in the private implementation repository.
