# KubeClaw Architecture

## Overview

KubeClaw diagnoses Kubernetes incidents from **frozen replay evidence** — pre-recorded outputs from a strict six-tool allowlist. The engine is fully deterministic: no LLM calls, no randomness, no live cluster access during diagnosis, and no write operations.

The repository documents a target contract boundary for a possible future AI
explanation layer. A strict `agent_safe_response/v1` envelope exists and
deterministic OpenClaw output is mapped and validated against it, but
real-agent admission is currently blocked because stable citations, output
policy enforcement, and adversarial evaluation are not complete. No real
autonomous AI agent currently exists.

![KubeClaw end-to-end architecture](../images/kubeclaw-diagram.png)

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

                  ┌──────────────────────────┐
                  │  OpenClaw Contract Layer │
                  │                          │
                  │  Request validation +    │
                  │  agent_safe_response/v1  │
                  │  mapped & validated      │
                  │  output envelope         │
                  └──────────────────────────┘
                         │
              Future constrained consumers
              may use this boundary only
              after admission hardening.
```

## Optional layers

Beyond the core engine, two offline prototype layers demonstrate parts of a
possible AI integration boundary:

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

The agent consumer and LLM adapter are **not** part of the engine. They do not
call kubectl or a real LLM and do not remediate. Their current string/value
checks are policy scaffolds, not sufficient real-agent gates.

## Key invariants

- **Replay evidence is the source of truth.** All diagnosis runs against frozen tool outputs. The engine does not call kubectl or access live clusters during replay.

- **Live Kubernetes is optional and local-only.** A dedicated kind lab can capture read-only evidence into generated replay sessions. Capture is not diagnosis. Generated sessions are unreviewed until promoted through corpus governance.

- **OpenClaw is a contract scaffold.** Request validation enforces the tool
  allowlist, blocks write verbs, validates request payloads, and rejects unknown
  tools or fields. Deterministic OpenClaw output can be mapped into the closed
  `agent_safe_response/v1` envelope and validated against it before being
  exposed; malformed envelopes fail closed. Semantic-policy, stable-citation,
  and adversarial-evaluation hardening remain required, and real-agent
  admission remains blocked.

- **Agent evaluation is separate from engine evaluation.** Engine evaluation measures classification accuracy, confidence calibration, and replay correctness. Agent evaluation measures whether an external agent preserves diagnoses, respects refusals, avoids invented facts, and avoids unsafe remediation.

- **Drift is bounded.** At most one drift recheck per session. Drift performs full recomputation from merged evidence — it never patches the previous answer. Post-drift confidence must not exceed pre-drift confidence.

## Evaluation corpora

| Corpus | Sessions | Purpose |
| --- | --- | --- |
| Golden | 20 | Development regression; CI replay |
| Holdout | 24 | Evaluation-only; engine behaviour must not be tuned against holdout results |
| Generated | — | Unreviewed live-capture fixtures; not evaluation material until reviewed and promoted |

The governed replay corpus totals 44 sessions. Session types covered include
crashloop, imagepull, oom, pending, service_unreachable, probe_failure,
config_dependency_failure, insufficient evidence, refusal, tool failure, and
drift.

## What is not in this showcase

This repository documents architecture, safety, and sample outputs only. It does not include the implementation source, test suite, calibration artifacts, or governed replay corpora. Those remain in the private implementation repository.
