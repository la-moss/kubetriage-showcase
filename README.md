# KubeTriage

**A safe, evidence-based way to investigate Kubernetes incidents.**

When a workload on Kubernetes fails, the first problem is usually not “what command should we run?” — it is “what actually happened, and can we prove it?”

KubeTriage investigates from a **frozen bundle of read-only evidence**: events, logs, and resource descriptions captured ahead of time. It extracts typed facts, classifies the failure, ranks hypotheses, scores confidence, and links conclusions back to exact evidence locations. The same evidence always produces the same result.

Think of it as an incident investigator working from a sealed evidence file:

- the evidence cannot change halfway through the investigation;
- the same case can be replayed;
- another person can inspect how the conclusion was reached;
- unsupported conclusions are rejected rather than presented as fact.

> KubeTriage is not an autonomous Kubernetes agent. It is the deterministic diagnostic authority that a future constrained explanation layer could sit behind.

KubeTriage itself is not AI: KubeTriage is not an AI Kubernetes agent, and no real autonomous AI agent exists. There is no LLM in the engine. There is no remediation and no cluster mutation path. Real-agent admission remains blocked.

![KubeTriage architecture](images/kubetriage-diagram.png)

---

## Why this project exists

AI models can produce clear and convincing explanations, but clarity is not the same as correctness. Near critical infrastructure, an explanation should not be allowed to invent evidence, change the diagnosed problem, inflate confidence, continue after a safety refusal, or turn a suggestion into a cluster action.

KubeTriage separates **diagnosis from explanation**. It requires grounded evidence, an authority boundary, and safe execution controls. The goal is not "AI runs kubectl". The deterministic engine decides what happened, how confident the diagnosis is, which facts support it, whether evidence is insufficient, and whether the request must be refused. A future explanation layer may describe an admitted result — it would not be allowed to change it.

---

## What it does

Given frozen read-only Kubernetes evidence, KubeTriage:

1. validates the request and redacts secrets, tokens, and sensitive material;
2. extracts typed facts with exact citations;
3. classifies against seven bounded incident classes;
4. ranks hypotheses and scores confidence from explicit evidence inputs;
5. optionally performs one bounded drift recheck when new evidence arrives;
6. seals an immutable run manifest and provenance bundle;
7. admits operator-visible explanations only after claim → fact → evidence verification.

Replay is the source of truth: no hidden state, no online learning, no randomness.

See [Architecture](docs/architecture.md) for the full data flow and layer separation.

---

## What it does not do

| KubeTriage is not | Boundary |
| --- | --- |
| An auto-remediator | No apply, patch, delete, restart, scale, or fixes. |
| An LLM diagnostic engine | No model calls or provider SDKs in the engine. |
| A live production service | Diagnosis runs on frozen replay evidence. |
| An autonomous AI agent | No real autonomous agent has been admitted. |
| Unrestricted Kubernetes coverage | Seven pattern-based classes; returns `insufficient_evidence` otherwise. |
| A replacement for an SRE | Evidence-backed reports; humans decide what to do next. |

Safety is enforced in code, not prompts. See [Safety model](docs/safety-model.md).

---

## Incident classes

KubeTriage currently supports seven top-level deterministic incident classes:

| Class | Meaning |
| --- | --- |
| `crashloop` | Container repeatedly fails and restarts |
| `imagepull` | Image cannot be pulled from the registry |
| `oom` | Container killed by the OOM killer |
| `pending` | Pod cannot be scheduled |
| `service_unreachable` | Service has no reachable endpoints |
| `probe_failure` | Readiness or liveness probe failing |
| `config_dependency_failure` | Missing or invalid ConfigMap or Secret dependency |

Two important non-diagnostic outcomes: **`insufficient_evidence`** (honest uncertainty) and **refusal** (safety violation — terminal, no further processing).

---

## Current status

**Latest reviewed baseline:** `v2.2.0-a6b-s2-reviewed-baseline` (A6B-S2 lifecycle ledger and idempotency).

| Area | Result |
| --- | --- |
| Golden replay (20 golden sessions) | **20/20** |
| Holdout replay (24 holdout sessions) | **24/24** |
| Governed corpus | **44 sessions** total |
| Adversarial evaluation (A5A) | **102/102** |
| Metamorphic evaluation (A5B) | **67/67** |
| Real-agent admission | **Blocked** |

The diagnostic engine (A4A/A4B) and adversarial evaluation programme (A5) are complete. **A6** controlled explanation-agent admission is in progress: **A6A** (admission constitution) and **A6B-S2** (lifecycle ledger) are published; **A6B-S3+** and **A6C–A6G** are not started. That progress does not connect a provider or admit an autonomous agent.

Deterministic OpenClaw output validation remains a boundary milestone, not real-agent admission. Earlier `agent_safe_response/v1` envelope work is superseded for the admitted path by provenance-aware `agent_explanation/v3` under `agent_output_policy/v3`.

Full gate table, evaluation numbers, frozen digests, and A6 roadmap: **[docs/current-status.md](docs/current-status.md)**.

---

## Try the samples

Representative outputs from the private implementation (no live cluster required):

### Core diagnostic demos

| Sample | Demonstrates |
| --- | --- |
| [Image-pull diagnosis](sample-output/engine-imagepull-demo.txt) | Normal deterministic diagnosis (`imagepull`, confidence 0.88) |
| [Drift class flip](sample-output/engine-drift-demo.txt) | Full drift recomputation; confidence does not inflate |
| [Safety refusal](sample-output/engine-refusal-demo.txt) | Terminal refusal for unsafe cluster context |
| [Combined session output](sample-output/demo-session-outputs.txt) | All three paths in one file |

### Provenance and evaluation (current admitted path)

| Sample | Demonstrates |
| --- | --- |
| [Provenance-aware explanation](sample-output/provenance-aware-explanation-demo.json) | Accepted `agent_explanation/v3` under `agent_output_policy/v3` |
| [Adversarial evaluation summary](sample-output/adversarial-evaluation-summary.json) | A5A corpus: 102/102, zero unexpected admissions |
| [Metamorphic evaluation summary](sample-output/metamorphic-evaluation-summary.json) | A5B suite v2: 67/67, pure vs mixed removal distinction |
| [Confidence decomposition](sample-output/confidence-decomposition-demo.json) | Reviewed mixed-information removal (0.72 → 0.88 explained mechanically) |

### Earlier boundary milestones (labelled legacy)

| Sample | Demonstrates |
| --- | --- |
| [Safe response envelope](sample-output/agent-safe-response-demo.json) | Earlier `agent_safe_response/v1` authority envelope — envelope-local IDs such as `ev-001` are **not** current A4B stable fact identity |
| [Deterministic consumer](sample-output/agent-consumer-demo.txt) | Offline `agent_output_policy/v2` scaffold — not the v3 provenance path |
| [LLM policy rejection](sample-output/llm-policy-demo.txt) | Offline policy scaffold — no real LLM is called |

Presenter script: **[docs/demo-walkthrough.md](docs/demo-walkthrough.md)**.

---

## How it works (short)

```text
frozen read-only evidence
  → validation and redaction
  → typed facts with exact citations
  → classification → hypothesis ranking → confidence
  → optional one bounded drift recheck (full recomputation)
  → run manifest + provenance
  → provenance-aware policy validation
  → deterministic operator-visible output
```

Six read-only logical tools only: `events_tail`, `describe_pod`, `describe_deploy`, `logs`, `get_yaml`, `top_pod`. Write verbs (`apply`, `patch`, `delete`, `exec`, …) are refused.

Details: [Architecture](docs/architecture.md) · [Safety model](docs/safety-model.md) · [Live fixture pipeline](docs/live-fixture-pipeline.md) (optional local kind capture).

---

## Honest limitations

- Seven pattern-based classes — not every Kubernetes failure mode.
- Small governed corpus (20 golden, 24 holdout, 44 sessions); calibration figures are not broad production claims.
- ECE remains above target and is documented, not tuned away.
- Deterministic integrity checking — not cryptographic authenticity.
- Implementation source, test suites, and replay corpora remain private.
- Real-agent admission remains blocked despite A5 completion and A6B-S2 publication.

The system returns `insufficient_evidence` rather than guessing when evidence does not justify a supported diagnosis.

---

## About this repository

This showcase documents purpose, architecture, safety boundaries, evaluation status, and representative outputs. It does not include implementation code, private test suites, governed replay corpora, raw cluster evidence, or provider integration.

| Document | Purpose |
| --- | --- |
| [Current status](docs/current-status.md) | Five-minute briefing after A6B-S2 reviewed baseline |
| [Architecture](docs/architecture.md) | Data flow, components, and invariants |
| [Safety model](docs/safety-model.md) | Read-only boundaries, refusals, and governance |
| [Demo walkthrough](docs/demo-walkthrough.md) | How to present KubeTriage in 3–10 minutes |
| [Live fixture pipeline](docs/live-fixture-pipeline.md) | Optional local kind capture workflow |

---

## In one sentence

> KubeTriage is a read-only Kubernetes incident investigator that works from frozen evidence, produces the same diagnosis every time, and can show exactly where every conclusion came from.
