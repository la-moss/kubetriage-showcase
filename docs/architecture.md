# KubeTriage Architecture

## Overview

KubeTriage diagnoses Kubernetes incidents from **frozen replay evidence** — pre-recorded outputs from a strict six-tool allowlist. The engine is fully deterministic: no LLM calls, no randomness, no live cluster access during diagnosis, and no write operations.

Deterministic diagnosis is the authority. Provenance and policy enforcement sit on top of that authority. Offline OpenClaw and LLM packages are scaffolds only. No provider boundary is admitted. Real-agent admission remains blocked even though A4A, A4B, A5A, and A5B have passed independent review and the A5 programme is complete.

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

## Evaluation corpora

| Corpus | Sessions | Purpose |
| --- | --- | --- |
| Golden | 20 | Development regression; CI replay |
| Holdout | 24 | Evaluation-only; never tuned against |
| Generated | — | Unreviewed live-capture fixtures; not evaluation material until promoted |

Session types include crashloop, imagepull, oom, pending, service_unreachable, probe_failure, config_dependency_failure, insufficient evidence, refusal, tool failure, and drift.

## What is not in this showcase

This repository documents architecture, safety, evaluation summaries, and sample outputs only. It does not include the implementation source, test suite, calibration artifacts, private adversarial fixtures, or governed replay corpora. Those remain in the private implementation repository.
