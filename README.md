# KubeTriage

**A deterministic diagnostic engine for cloud-native systems.**

KubeTriage transforms **governed operational evidence** into **replayable, evidence-backed diagnoses**. It validates input, extracts typed facts, evaluates hypotheses, calculates confidence, and records provenance so every conclusion stays traceable to the evidence that produced it. The same evidence always produces the same result.

The current implementation begins with **Kubernetes incident triage**. The architecture is deliberately centred on **deterministic diagnosis** — not on a fixed set of failure types or a single evidence source.

> KubeTriage is not an autonomous agent. It is the deterministic diagnostic authority that a future constrained explanation layer could sit behind.

KubeTriage itself is not AI: KubeTriage is not an AI Kubernetes agent. No autonomous agent is currently connected to or admitted by KubeTriage. There is no LLM in the engine. There is no remediation and no cluster mutation path. Real-agent admission remains blocked.

![KubeTriage overview — current Kubernetes implementation](images/kubetriage-overview.png)

---

## Why this project exists

Production incidents rarely fail on a single signal. Engineers often have to correlate workload state, configuration, events, logs, and related observations from multiple places before they can defend a conclusion.

That manual process is slow to replay and easy to dispute: different people can reach different explanations from the same incident, and a confident narrative is not necessarily a correct one. Near critical infrastructure, an explanation should not invent evidence, change the diagnosed problem, inflate confidence, continue after a safety refusal, or turn a suggestion into a cluster action.

KubeTriage is intended to make diagnostic reasoning:

- **bounded** — operate only on governed evidence and supported classes;
- **reproducible** — same evidence, same diagnosis;
- **inspectable** — conclusions link to exact evidence locations;
- **evidence-backed** — unsupported conclusions are rejected;
- **confidence-aware** — confidence is derived from explicit evidence inputs.

KubeTriage separates **diagnosis from explanation**. Diagnosis is deterministic and evidence-grounded. A future explanation layer may describe an admitted result — it would not be allowed to change it. The goal is not "AI runs kubectl". It requires grounded evidence, an authority boundary, and safe execution controls.

---

## Observability vs diagnosis

Observability systems are good at showing what changed, where failures appeared, which metrics moved, which components became unhealthy, and what network or application behaviour was observed.

KubeTriage addresses a different question:

> Given the available evidence, what is the most likely explanation, how confident is that conclusion, and exactly which evidence supports it?

KubeTriage is **not** intended to replace observability systems. Its role is to **consume governed evidence** and turn that evidence into deterministic, replayable diagnosis. Humans remain responsible for deciding and performing remediation.

---

## Where KubeTriage fits

KubeTriage sits in the **diagnostic reasoning layer**:

```text
operational evidence
  → validation
  → fact extraction
  → hypothesis evaluation
  → confidence
  → provenance
  → replayable diagnosis
```

Modern incidents often require correlating multiple types of evidence. Potential evidence domains include workload and resource state, configuration, Kubernetes events, logs, service and endpoint state, networking evidence, DNS, identity, policy, deployment history, and infrastructure state.

Those domains are examples of the kinds of operational evidence a diagnostic engine may need to reason over. They are **not** a claim that all are currently implemented.

KubeTriage reasons over **governed evidence** rather than generating conclusions from opaque model reasoning. Diagnosis and explanation remain separate concerns. Additional evidence sources, if ever admitted, must preserve determinism, provenance, replayability, and bounded authority.

See [Architecture](docs/architecture.md) for the full data flow and layer separation.

---

## Current implementation

What exists today, as proven by the governed Kubernetes replay path:

- Frozen, read-only **Kubernetes-native** evidence via a strict six-tool allowlist (`events_tail`, `describe_pod`, `describe_deploy`, `logs`, `get_yaml`, `top_pod`).
- Typed fact extraction with exact citations.
- Seven bounded incident classes (see below).
- Hypothesis ranking and confidence scoring from explicit evidence inputs.
- At most one bounded drift recheck with full recomputation.
- Immutable run manifest and claim → fact → evidence provenance.
- Provenance-aware explanation admission under `agent_output_policy/v3` (real-agent admission still blocked).
- Offline evaluation against golden, holdout, adversarial, and metamorphic corpora.

Given that frozen Kubernetes evidence, KubeTriage:

1. validates the request and redacts secrets, tokens, and sensitive material;
2. extracts typed facts with exact citations;
3. classifies against seven bounded incident classes;
4. ranks hypotheses and scores confidence from explicit evidence inputs;
5. optionally performs one bounded drift recheck when new evidence arrives;
6. seals an immutable run manifest and provenance bundle;
7. admits operator-visible explanations only after claim → fact → evidence verification.

Replay is the source of truth: no hidden state, no online learning, no randomness.

### Supported incident classes (current)

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

These seven classes are the current proof point. They are not the permanent architectural boundary of the diagnostic engine.

---

## Architectural scope

The architecture is designed to accommodate additional **governed operational evidence sources** over time while preserving:

- determinism and replayability;
- exact provenance from conclusion to evidence;
- bounded authority (diagnosis remains engine-owned);
- separation of diagnosis from explanation.

Potential governed evidence sources may include networking, DNS, identity, policy, deployment history, and broader infrastructure state — always as frozen, reviewed inputs, never as live opaque telemetry streams inside the diagnostic authority.

This section describes architectural direction only. It does **not** mean those integrations exist, are scheduled, or are committed.

---

## What it does not do

| KubeTriage is not | Boundary |
| --- | --- |
| An observability, monitoring, or telemetry platform | It consumes governed evidence; it does not collect or store operational telemetry. |
| An auto-remediator | No apply, patch, delete, restart, scale, or fixes. Humans decide remediation. |
| An LLM diagnostic engine | No model calls or provider SDKs in the engine. |
| A live production service | Diagnosis runs on frozen replay evidence. |
| An autonomous AI agent | No autonomous agent is connected to or admitted by KubeTriage. |
| Unrestricted incident coverage | Seven pattern-based classes today; returns `insufficient_evidence` otherwise. |
| A multi-source evidence platform today | Broader evidence domains are architectural scope, not current capability. |

Safety is enforced in code, not prompts. See [Safety model](docs/safety-model.md).

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

Six read-only logical tools only (current implementation): `events_tail`, `describe_pod`, `describe_deploy`, `logs`, `get_yaml`, `top_pod`. Write verbs (`apply`, `patch`, `delete`, `exec`, …) are refused.

Details: [Architecture](docs/architecture.md) · [Safety model](docs/safety-model.md) · [Live fixture pipeline](docs/live-fixture-pipeline.md) (optional local kind capture).

---

## Honest limitations

Distinguish **current implementation constraints** from **architectural intent**:

| Architectural intent | Implemented today |
| --- | --- |
| Deterministic, evidence-grounded diagnosis with provenance and replay | Yes — for governed Kubernetes-native replay evidence |
| Diagnosis separated from explanation | Yes — real-agent explanation admission remains blocked |
| Multiple governed operational evidence sources | Architectural direction only — not implemented |
| Constrained explanation consumer behind the diagnostic authority | A6 in progress; no provider connected; real-agent admission blocked |

Current implementation constraints:

- Seven pattern-based Kubernetes classes — not every Kubernetes failure mode, and not a claim of cloud-native coverage beyond that proof point.
- Broader multi-source diagnosis (networking, identity, policy, infrastructure state, external observability feeds) remains architectural direction, not current capability.
- Small governed corpus (20 golden, 24 holdout, 44 sessions); calibration figures are not broad production claims.
- ECE remains above target and is documented, not tuned away.
- Deterministic integrity checking — not cryptographic authenticity.
- Implementation source, test suites, and replay corpora remain private.
- Real-agent admission remains blocked despite A5 completion and A6B-S2 publication.
- No arbitrary telemetry ingestion, live autonomous diagnosis, remediation, or LLM diagnostic authority.

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

> KubeTriage is a deterministic evidence engine for cloud-native incident diagnosis: it turns governed operational evidence into replayable, evidence-backed conclusions — today proven on Kubernetes-native replay inputs.
