# KubeTriage

**Safe, explainable incident triage for Kubernetes.**

KubeTriage is a deterministic Kubernetes diagnostic engine built around one principle:

> Evidence should decide the diagnosis. An AI explanation layer should never be allowed to override it.

It reads frozen, read-only Kubernetes evidence, extracts typed facts, classifies supported incident types, ranks hypotheses, computes confidence, and produces reproducible reports. It does not remediate incidents, mutate clusters, call an LLM, or execute arbitrary commands.

## Current status

The deterministic hardening and adversarial evaluation programme is complete through **A5**.

| Gate | Status |
| --- | --- |
| Golden replay corpus | **20/20 passed** |
| Holdout replay corpus | **24/24 passed** |
| A4A — run manifest and evidence identity | **Passed independent review** |
| A4B — exact claim → fact → evidence provenance | **Passed independent review through H4** |
| A5A — governed adversarial admission corpus | **102/102 passed** |
| A5B — property and metamorphic evaluation | **67/67 passed** |
| Unexpected adversarial admissions | **0** |
| Capability leaks | **0** |
| Renderer leaks | **0** |

Real-agent admission remains **blocked**. There is no provider integration, no autonomous AI agent, no remediation, no live production diagnosis mode, and no cluster mutation path.

## Why I built this

AI agents near Kubernetes create a particular risk: a model can sound certain while having no grounded evidence for its diagnosis, no mechanical link between its prose and the system state, and no safe boundary between a suggestion and a cluster action.

KubeTriage starts from the opposite direction.

The deterministic diagnostic authority comes first:

```text
frozen evidence
→ typed facts
→ deterministic classification
→ ranked hypotheses
→ confidence
→ exact evidence provenance
→ constrained operator-visible report
```

A future AI layer could explain an admitted result, but it would not be allowed to change:

- the diagnosed class;
- the confidence;
- the refusal or insufficient-evidence outcome;
- the drift result;
- the supporting facts;
- the evidence behind those facts.

## What KubeTriage does

Given a frozen bundle of read-only Kubernetes tool outputs, KubeTriage:

1. validates and redacts the evidence;
2. extracts typed facts with exact extraction-time citations;
3. classifies the incident deterministically;
4. ranks possible explanations;
5. computes confidence from explicit evidence inputs;
6. optionally performs one bounded drift recheck;
7. produces JSON and Markdown reports;
8. verifies a complete claim → fact → evidence chain before deterministic rendering.

The same evidence produces the same result. There is no hidden state, online learning, randomness, or wall-clock dependency.

## What it does not do

| Not this | Current boundary |
| --- | --- |
| Auto-remediator | It never applies, patches, deletes, scales, restarts, or executes fixes. |
| LLM diagnostic engine | The engine contains no model calls. |
| Kubernetes operator | It does not reconcile or mutate resources. |
| Shell wrapper | Arbitrary shell execution is not permitted. |
| Live production diagnosis service | Replay evidence is the source of truth. |
| Unrestricted AI agent | No provider or autonomous agent is admitted. |

KubeTriage is the deterministic safety substrate a future constrained agent would need to sit behind.

## Incident coverage

KubeTriage currently supports seven top-level deterministic incident classes:

| Class | Description |
| --- | --- |
| `crashloop` | Container repeatedly fails and restarts |
| `imagepull` | Image cannot be pulled from a registry |
| `oom` | Container was killed by the OOM killer |
| `pending` | Pod cannot be scheduled |
| `service_unreachable` | Service has no reachable endpoints |
| `probe_failure` | Readiness or liveness probe failure |
| `config_dependency_failure` | Missing or invalid ConfigMap or Secret dependency |

Special outcomes:

- `insufficient_evidence` — the evidence does not support a confident single-class diagnosis;
- `refusal` — the request violates the safety constitution or execution boundary.

This is intentionally not complete Kubernetes incident coverage. Unsupported or ambiguous cases return `insufficient_evidence` rather than forcing a diagnosis.

## Safety model

Safety is enforced in code, not prompts:

- strict six-tool allowlist;
- read-only execution boundary;
- context and namespace guards;
- blocked write and mutation verbs;
- spoofing, homoglyph and schema-deviation checks;
- secret, token and PEM redaction;
- bounded action and fact budgets;
- terminal refusals;
- one bounded drift recomputation;
- no arbitrary shell execution;
- no LLM in the diagnostic engine.

## Exact evidence provenance

KubeTriage can mechanically verify this chain:

```text
rendered statement
→ constrained claim
→ replayed classifier or hypothesis contribution
→ identity-bound support set
→ typed fact
→ exact JSON Pointer or UTF-8 byte range
→ verified redacted evidence artifact
→ sealed run manifest
```

The provenance boundary includes:

- stable evidence-artifact identity;
- stable typed-fact identity;
- extraction-time citations rather than later text search;
- UTF-8-safe byte ranges;
- typed-value verification against selected evidence;
- replayed classifier contributions;
- independently recomputed contribution and support-set identities;
- no fallback to a weaker policy after provenance failure;
- renderer access only through an immutable validated capability.

This is deterministic replay verification, not cryptographic authenticity or signed storage.

## Evaluation

### Replay corpus

The governed replay corpus contains **44 sessions**:

- **20 golden sessions** for development regression;
- **24 holdout sessions** for evaluation only.

Latest results:

| Metric | Result |
| --- | ---: |
| Golden replay | 20/20 |
| Holdout replay | 24/24 |
| Base top-1 accuracy | 100.0% |
| Brier score | 0.0557 |
| ECE | 0.2186 |
| False high confidence | 0 |
| Drift flip accuracy | 100.0% |
| Drift non-inflation violations | 0 |

ECE remains above target on a small corpus and is reported rather than tuned away.

### A5A — governed adversarial admission corpus

- **102/102 cases passed**
- unexpected admissions: **0**
- capability leaks: **0**
- renderer leaks: **0**
- frozen corpus digest: `60504978eaa046f9fe04b4c532ebf4c6039c74e2e42caa727312f384e370bb52`

The corpus covers authority, manifest, artifact, pointer, fact, contribution, support-set, claim, drift, cross-run, capability, renderer, and coherent multi-step attacks.

### A5B — property and metamorphic evaluation

- **67/67 properties passed**
- capability leaks: **0**
- renderer leaks: **0**
- nondeterministic repeats: **0**
- frozen suite digest: `b367b02266444c0b680b28fb97ae60c197e4e00c2673aac3285fb82fcaebd124`

The suite tests:

- ordering invariance;
- duplicate evidence amplification;
- irrelevant noise;
- operational metadata changes;
- canonicalisation;
- supporting-evidence removal;
- truncation;
- strengthening and conflicting evidence;
- identity sensitivity;
- refusal and insufficient-evidence safety;
- drift behaviour;
- repeat determinism;
- capability sealing;
- multi-step transformations.

Confidence-removal evaluation distinguishes:

- **pure support removal** — confidence must not increase;
- **mixed-information removal** — support and confidence-dampening inputs changed together, so the exact confidence decomposition must explain the result.

## Demo flow

```text
controlled local kind workload
→ read-only evidence capture
→ governed replay session
→ deterministic diagnosis
→ sealed authority and exact provenance
→ constrained deterministic explanation
```

Most demos run fully offline against frozen replay evidence. No live cluster is required for diagnosis, testing, calibration, or adversarial evaluation.

![KubeTriage architecture — deterministic Kubernetes diagnostics with safety by design](images/kubeclaw-diagram.png)

## Sample outputs

| File | Demonstrates |
| --- | --- |
| [engine-imagepull-demo.txt](sample-output/engine-imagepull-demo.txt) | Normal deterministic diagnosis |
| [engine-drift-demo.txt](sample-output/engine-drift-demo.txt) | Drift recomputation and class flip |
| [engine-refusal-demo.txt](sample-output/engine-refusal-demo.txt) | Terminal safety refusal |
| [agent-safe-response-demo.json](sample-output/agent-safe-response-demo.json) | Closed deterministic authority envelope |
| [agent-consumer-demo.txt](sample-output/agent-consumer-demo.txt) | Constrained offline consumer shape |
| [llm-policy-demo.txt](sample-output/llm-policy-demo.txt) | Offline policy-boundary demonstration |

## Real-agent readiness

The project now has a hardened deterministic authority and evaluation boundary, but that does not make it ready for unrestricted agent use.

Remaining gates are operational and human-governed:

- provider and model isolation;
- authentication and authorisation;
- tenant and credential boundaries;
- request lifecycle and idempotency;
- concurrency and resource limits;
- audit retention and trace correlation;
- admission revocation after regressions;
- human-reviewed shadow operation;
- explicit approval and kill-switch procedures.

No real AI agent, provider SDK, remediation path, or cluster-mutation capability exists in this project.

## Known limitations

- Seven pattern-based incident classes, not complete Kubernetes coverage.
- Replay is the diagnostic source of truth; there is no production live-diagnosis service.
- The corpus is deliberately small, limiting statistical calibration claims.
- ECE remains above target on the current holdout corpus.
- Tool-failure facts fail closed for evidence-backed explanation admission.
- Evaluation covers governed known attacks and declared transformations; it is not formal verification.
- No cryptographic signing or authenticity layer.
- Real-agent admission remains blocked.

## Repository note

The full implementation repository remains private while operational admission work is still under development. This public showcase documents the architecture, safety boundary, evaluation results, and representative outputs.

## Documentation

| Document | Purpose |
| --- | --- |
| [Architecture](docs/architecture.md) | Data flow, invariants and component overview |
| [Demo walkthrough](docs/demo-walkthrough.md) | Guided 3–10 minute demonstration |
| [Live fixture pipeline](docs/live-fixture-pipeline.md) | Optional local kind lab and read-only capture |
| [Safety model](docs/safety-model.md) | Safety boundaries, guards and governance rules |

---

**KubeTriage does not try to make Kubernetes autonomous. It tries to make diagnosis reproducible, bounded, and difficult to fake.**