# KubeTriage

KubeTriage is a deterministic Kubernetes diagnostic engine designed around safety-first infrastructure automation.

> KubeTriage is not an AI Kubernetes agent. It is a deterministic safety substrate that a future AI infrastructure agent could consume, once additional admission gates pass.

## Why I built this

AI agents near Kubernetes are risky in a specific way: a language model can sound confident while having no grounded evidence for its claims, no authority boundary separating its prose from the actual diagnosis, and no safe execution controls between its output and a cluster. An agent that hallucinates a root cause is bad; one that hallucinates a fix and runs it is worse.

KubeTriage explores the opposite approach. Instead of starting with an AI agent and bolting safety on afterwards, it builds the deterministic diagnostic authority layer first — before any AI explanation layer is admitted. The engine decides what is diagnosed, at what confidence, from which evidence; any future AI layer would only be allowed to explain that result, never to override it.

The goal is not "AI runs kubectl". The goal is a replayable, governed, citation-ready diagnostic substrate that a future constrained AI agent could consume:

- **Replayable** — every diagnosis is reproducible from frozen evidence, so claims can be checked.
- **Governed** — the evaluation corpus grows only through human review and explicit promotion rules.
- **Citation-ready** — evidence is typed and identified so an explanation layer can be forced to cite rather than invent.

Even with that substrate in place, real-agent admission remains blocked until A3 policy hardening, A4 stable evidence citation/provenance hardening, A5 adversarial evaluation, operational isolation review, and human approval gates pass.

KubeTriage is not the AI agent. It is the deterministic boundary a future AI infrastructure agent would need to sit behind.

## What it is

- A **deterministic** diagnostic engine: the same frozen evidence always produces the same classification, confidence, hypothesis ranking, and report.
- **Replay-first**: all diagnosis, testing, and evaluation run against frozen, reviewed replay evidence. No live cluster is required.
- **Safety-first**: read-only by design, with context/namespace guards, a strict six-tool allowlist, redaction, and terminal refusals — all enforced in code, not prompts.
- A **governed evaluation project**: golden and holdout corpora are managed under explicit governance rules, and calibration is reported honestly.
- The deterministic boundary layer that a future, constrained AI explanation agent would need to sit behind.

## What it is not

**KubeTriage itself is not AI.**

- It is not an LLM and contains **no LLM in the engine** — engine logic is deterministic code, not model inference.
- It is not an autonomous Kubernetes agent. **No real autonomous AI agent exists** in this project.
- It performs **no remediation** — all write and mutation verbs are refused; it never patches workloads or executes fixes.
- It has **no live diagnosis mode** — replay evidence is the source of truth; live capture is an optional local-lab fixture pipeline only.
- It does not let an LLM execute kubectl, and it makes no LLM/provider calls.
- **Real-agent admission remains blocked**: an AI-agent boundary and a strict response envelope are documented and validated (see below), but the remaining policy, citation/provenance, and adversarial-evaluation gates are still deferred.

| Not this | Why |
| --- | --- |
| A remediation tool | Strictly read-only. All write verbs are refused. |
| An autonomous operator | No cluster mutation; the engine never invokes kubectl. |
| A production live diagnosis system | No live diagnosis mode exists. Replay evidence is the source of truth; live capture is local-lab fixture generation only. |
| An LLM agent | Engine logic is deterministic code, not model inference. No LLM/provider is called. |
| Ready for real-agent admission | No. Admission remains blocked pending A3 policy hardening, A4 citation/provenance hardening, and A5 adversarial evaluation, plus human approval gates. |
| Complete Kubernetes incident coverage | No. Seven pattern-based top-level classes are covered; everything else returns `insufficient_evidence`. |
| A replacement for human SREs | Produces reports and refusals; humans decide what to do next. |

Given frozen read-only evidence from a bounded set of diagnostic tools, KubeClaw validates and redacts inputs, extracts facts, classifies incidents, ranks hypotheses, computes calibrated confidence, optionally performs one bounded drift recheck, and produces reproducible reports.

![KubeTriage architecture — deterministic Kubernetes diagnostics with safety by design](images/kubeclaw-diagram.png)

---

## Current coverage

KubeTriage currently supports **seven top-level deterministic incident classes**:

| Class | Description |
| --- | --- |
| `crashloop` | Container repeatedly fails and restarts |
| `imagepull` | Image cannot be pulled from registry |
| `oom` | Container killed by OOM killer |
| `pending` | Pod cannot be scheduled |
| `service_unreachable` | Service has no reachable endpoints |
| `probe_failure` | Readiness/liveness probe failure (grouped) |
| `config_dependency_failure` | Missing/invalid ConfigMap or Secret dependency (grouped) |

Special outcomes: `insufficient_evidence` (a valid result when evidence does not support a confident single-class conclusion) and **refusal** (a terminal safety outcome).

This is deliberately **not complete Kubernetes incident coverage**. The seven classes are limited, pattern-based coverage of common failure modes. Multi-component failures, application-layer errors, persistent volume issues, admission controller rejections, and cluster-level network problems are not classified; the engine returns `insufficient_evidence` rather than guessing.

---

## Evaluation status

The governed replay corpus currently contains **44 sessions**: **20 golden** (development regression) and **24 holdout** (evaluation-only, never tuned against).

Latest validation state of the private implementation repository:

| Check | Result |
| --- | --- |
| Test suite (pytest) | 1201 passed, 1 skipped |
| Golden replay | 20/20 passed |
| Holdout replay | 24/24 passed |
| Calibration false high confidence (≥ 0.80) | 0 |
| Drift confidence non-inflation violations | 0 |

Calibration is evaluated against the holdout corpus with base (non-drift) and drift metrics reported separately, never merged into a single headline metric. No confidence tuning was performed against holdout results. ECE remains above its target on this small corpus — a documented small-corpus limitation, not a tuned-away number (see Known limitations).

### Expansion pattern

Coverage grows through a proven, repeatable, human-governed loop:

```
controlled kind lab
  → read-only evidence capture
  → generated replay session
  → deterministic classifier support
  → governed golden/holdout promotion
  → calibration check
```

This loop has been applied end-to-end twice, adding `probe_failure` and then `config_dependency_failure` as governed top-level classes. Generated captures are never auto-promoted; every corpus addition passes human review, redaction, and governance rules.

---

## Safety model

Safety is enforced in code, not prompts:

- **Read-only** — all write and mutation verbs (`apply`, `patch`, `delete`, `exec`, `scale`, …) are blocked in the runner.
- **Six-tool allowlist** — `events_tail`, `describe_pod`, `describe_deploy`, `logs`, `get_yaml`, `top_pod`; spoofed and unknown tools are rejected.
- **Context and namespace guards** — non-lab cluster contexts and protected namespaces are refused.
- **Terminal refusals** — after a refusal, no diagnostic loop continues and no remediation is attempted.
- **Redaction** — secret data, tokens, and PEM material are redacted from evidence.
- **Drift bound** — at most one drift recheck per session, always a full recomputation; confidence does not inflate after drift.
- **Governed corpora** — golden and holdout are governed; generated captures are unreviewed until promoted.

See [docs/safety-model.md](docs/safety-model.md) for the full model. These properties describe the deterministic engine and runner; they are not a claim that every future agent-output path is already enforced.

---

## AI-agent readiness

Where the project currently stands on the path toward a possible future constrained AI explanation layer:

- **AI-agent boundary documented (A1).** The target authority, evidence, and output-policy boundary is written down: an admitted agent may explain but never override, invent, mutate, or remediate.
- **Strict agent-safe envelope exists (A2A).** A closed `agent_safe_response/v1` contract defines what a future agent may consume (see below).
- **OpenClaw output can be mapped and validated (A2B).** Deterministic OpenClaw replay output is mapped into the safe envelope and validated against the contract before being exposed. Malformed envelopes fail closed.
- **Real-agent admission remains blocked.** A3 policy enforcement hardening, A4 stable evidence citation/provenance hardening, and A5 adversarial evaluation remain future gates, along with operational isolation review and explicit human approval.

No real AI agent exists, no LLM/provider is called, no remediation is performed, and no live diagnosis mode exists. The documented boundary is a hardening target, not a fully enforced runtime guarantee — safety bypass by a future agent is treated as a risk to be engineered against, not something claimed to be already ruled out. OpenClaw/LLM policy enforcement is deliberately described as incomplete.

### Safe response envelope

`agent_safe_response/v1` is a closed, versioned contract for deterministic KubeClaw output that a future agent would consume:

- **Exactly one outcome** per response: `diagnosis`, `insufficient_evidence`, or `refusal` — mutually exclusive, with inactive outcomes explicitly `null`.
- **Required drift, evidence, safety, and provenance sections** — drift state with an exact before/after delta when triggered; typed evidence statements with identifiers and source references; fixed safety assertions (read-only, no remediation, no live diagnosis, no LLM in engine, no kubectl mutation); and session/corpus/review provenance.
- **No remediation or command fields** — the schema has no place for fixes, action plans, executable commands, or agent-assigned confidence. Every object is closed (`additionalProperties: false`).
- **AI may explain but not override** — the envelope is an immutable authority record; agent prose would live structurally outside it.

Today this envelope validates deterministic OpenClaw-facing output only. It is a prerequisite for real-agent admission, not the admission itself.

A complete example envelope is included in this repository:
[sample-output/agent-safe-response-demo.json](sample-output/agent-safe-response-demo.json).
It shows a validated `imagepull` diagnosis with deterministic evidence IDs
(`ev-001`, …), the fixed safety assertions, and replay provenance. The sample
demonstrates the deterministic authority envelope; it is not real-agent
admission and contains no remediation or command fields — the schema has none.

---

## Known limitations

Honest limits of the current state:

- **Limited incident class coverage** — seven pattern-based top-level classes; not complete Kubernetes incident coverage.
- **Small but governed corpus** — 44 replay sessions total; small corpora limit statistical claims even under strict governance.
- **ECE remains imperfect on the small corpus** — Expected Calibration Error is above its target; the confidence tiers cluster in two buckets and single sessions dominate bins. Constants were not tuned to hide this.
- **A4-grade stable citations still deferred** — evidence identifiers are envelope-local; stable, machine-checkable citation identity and provenance hardening remain future work.
- **A3 policy enforcement hardening still deferred** — current OpenClaw/LLM policy checks are deterministic string/value scaffolds, not sufficient real-agent gates.
- **No real AI agent admitted** — real-agent admission remains blocked; no LLM/provider integration exists.
- **No remediation** — KubeClaw never executes or recommends fixes; humans decide what to do next.

The private implementation repository tracks the full list in its known-limitations document.

---

## Next hardening steps

Planned, in order — none of these are current guarantees:

1. **A3 — policy enforcement hardening**: exact preservation of `top_class`, confidence, refusal, and drift across any explanation boundary; fail-closed handling of policy-invalid output; remediation-policy hardening beyond known command strings.
2. **A4 — stable evidence citation and provenance hardening**: stable citation identifiers and mechanical rejection of uncited, altered, or unsupported claims.
3. **A5 — adversarial evaluation**: prompt injection, status omission, class substitution, confidence manipulation, drift suppression, and refusal continuation testing.
4. **Human approval gates** before any real agent or provider integration is considered.

---

## Current demo flow

```
local kind cluster
  → controlled broken workload
  → read-only evidence capture
  → generated replay session
  → deterministic diagnosis
  → validated agent_safe_response/v1 envelope (OpenClaw output validation)
  → optional deterministic agent explanation scaffold
  → optional LLM policy boundary scaffold
```

Most demos run fully offline against frozen replay evidence. No live cluster is required for diagnosis, evaluation, or the sample outputs in this repository.

See [docs/demo-walkthrough.md](docs/demo-walkthrough.md) for a guided walkthrough and [sample-output/](sample-output/) for representative engine and agent outputs.

---

## Repository note

The full implementation repository is currently private while the project is under active development. This showcase repo documents the architecture, safety model, current status, and demo outputs.

---

## Documentation

| Document | Purpose |
| --- | --- |
| [docs/architecture.md](docs/architecture.md) | Data flow, invariants, and component overview |
| [docs/demo-walkthrough.md](docs/demo-walkthrough.md) | How to present KubeTriage in 3–10 minutes |
| [docs/live-fixture-pipeline.md](docs/live-fixture-pipeline.md) | Optional local kind lab and read-only capture pipeline |
| [docs/safety-model.md](docs/safety-model.md) | Safety boundaries, guards, and governance rules |

## Sample outputs

| File | Demonstrates |
| --- | --- |
| [sample-output/engine-imagepull-demo.txt](sample-output/engine-imagepull-demo.txt) | Normal deterministic diagnosis |
| [sample-output/engine-drift-demo.txt](sample-output/engine-drift-demo.txt) | Drift recomputation and class flip |
| [sample-output/engine-refusal-demo.txt](sample-output/engine-refusal-demo.txt) | Safety refusal (terminal) |
| [sample-output/agent-safe-response-demo.json](sample-output/agent-safe-response-demo.json) | Validated `agent_safe_response/v1` diagnosis envelope (authority record, not real-agent admission) |
| [sample-output/agent-consumer-demo.txt](sample-output/agent-consumer-demo.txt) | OpenClaw agent explanation layer |
| [sample-output/llm-policy-demo.txt](sample-output/llm-policy-demo.txt) | LLM adapter prompt preview and policy rejection |
