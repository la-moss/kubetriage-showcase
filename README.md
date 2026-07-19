# KubeTriage

**A safe, evidence-based way to investigate Kubernetes incidents.**

When an application running on Kubernetes breaks, the first challenge is working
out what actually happened.

Logs, events and configuration can point in different directions. A confident
explanation is not necessarily a correct one, and an incorrect diagnosis becomes
more dangerous when software is also allowed to act on it.

KubeTriage investigates incidents from a frozen set of read-only evidence. It
extracts the relevant facts, identifies the most likely failure, records its
confidence, and shows which evidence supports its conclusions.

Think of it as an incident investigator working from a sealed evidence file:

- the evidence cannot change halfway through the investigation;
- the same case can be replayed;
- another person can inspect how the conclusion was reached;
- unsupported conclusions are rejected rather than presented as fact.

KubeTriage does not fix the incident, run commands against the cluster or allow
an AI model to decide the diagnosis. It is deliberately read-only and
deterministic: **the same evidence produces the same result every time.**

> KubeTriage is not an autonomous Kubernetes agent. It is the deterministic
> diagnostic authority that a future constrained explanation layer could sit
> behind.

---

## Why this project exists

AI models can produce clear and convincing explanations, but clarity is not the
same as correctness.

Near critical infrastructure, an explanation should not be allowed to:

- invent evidence;
- change the diagnosed problem;
- increase the confidence;
- continue after a safety refusal;
- turn a suggestion into a cluster action;
- hide where its claims came from.

KubeTriage separates diagnosis from explanation.

The deterministic engine decides:

- what happened;
- how confident the diagnosis is;
- which facts support it;
- whether the evidence is insufficient;
- whether the request must be refused.

A future AI layer may be allowed to explain an admitted result, but it would not
be allowed to change it.

---

## What KubeTriage does

Given a frozen bundle of read-only Kubernetes evidence, KubeTriage:

1. validates the request and the supplied evidence;
2. redacts secrets, tokens and other sensitive material;
3. extracts typed facts from events, logs and resource descriptions;
4. compares those facts against supported incident types;
5. ranks the most likely explanations;
6. calculates confidence from explicit evidence inputs;
7. optionally performs one bounded drift recheck when new evidence appears;
8. produces a structured report;
9. links every constrained conclusion back to verified facts and exact evidence.

Because the evidence is frozen, the investigation can be replayed exactly.

There is no hidden state, online learning, randomness or wall-clock dependency.

---

## What KubeTriage does not do

| KubeTriage is not | Current boundary |
| --- | --- |
| An auto-remediator | It does not apply, patch, delete, restart, scale or execute fixes. |
| A Kubernetes operator | It does not reconcile resources or change cluster state. |
| An LLM diagnostic engine | The diagnosis engine contains no model calls or provider SDKs. |
| A shell wrapper | Arbitrary commands and shell execution are not permitted. |
| A live production diagnostic service | Replay evidence remains the diagnostic source of truth. |
| An autonomous AI agent | No real autonomous agent has been admitted. |
| Complete Kubernetes incident coverage | It supports seven bounded incident classes and returns `insufficient_evidence` for unsupported cases. |
| A replacement for an SRE | It provides evidence-backed reports; a human decides what to do next. |

Safety is enforced in code, not prompts.

---

## Current diagnostic coverage

KubeTriage currently supports seven top-level incident classes:

| Class | What it means |
| --- | --- |
| `crashloop` | A container repeatedly fails and restarts |
| `imagepull` | A container image cannot be downloaded from the registry |
| `oom` | A container has been killed because it ran out of memory |
| `pending` | A pod cannot be scheduled |
| `service_unreachable` | A service has no reachable endpoints |
| `probe_failure` | A readiness or liveness health check is failing |
| `config_dependency_failure` | A required ConfigMap or Secret is missing or invalid |

KubeTriage also has two important non-diagnostic outcomes:

- `insufficient_evidence` — the evidence does not justify a confident conclusion;
- `refusal` — the request violates a safety rule and processing stops.

This is deliberately limited coverage. KubeTriage does not guess when a failure
does not match one of the supported classes.

---

## How it works

```text
Frozen read-only evidence
        ↓
Validation and redaction
        ↓
Typed fact extraction
        ↓
Deterministic classification
        ↓
Hypothesis ranking
        ↓
Confidence calculation
        ↓
Optional bounded drift recheck
        ↓
Evidence-backed report
        ↓
Provenance and policy validation
        ↓
Deterministic operator-visible output
```

The engine follows an enforced state machine:

```text
SIGNAL_INGESTION
→ FACT_EXTRACTION
→ CLASSIFICATION
→ HYPOTHESIS_RANKING
→ CONFIDENCE_SCORING
→ STOP_OR_NEXT
→ [DRIFT_RECHECK → FACT_EXTRACTION → …]
→ FINAL_REPORT
```

A drift recheck is a complete recomputation from the updated evidence. KubeTriage
does not patch the previous answer or allow confidence to inflate after drift.

![KubeTriage architecture](images/kubeclaw-diagram.png)

---

## Replay is the source of truth

Most diagnostic systems are tested against live environments that continue to
change while the test is running.

KubeTriage takes a different approach.

Each incident is represented by a frozen evidence bundle containing the outputs
of a bounded set of read-only diagnostic tools. The same incident can then be
replayed repeatedly.

This makes it possible to ask:

- Does the same evidence still produce the same diagnosis?
- Did a code change alter confidence?
- Did a harmless ordering change affect the answer?
- Can a forged fact or swapped evidence file bypass validation?
- Does removing evidence make the system incorrectly more confident?

Replay makes those questions testable.

---

## Read-only safety boundary

KubeTriage permits six logical diagnostic tools:

- `events_tail`
- `describe_pod`
- `describe_deploy`
- `logs`
- `get_yaml`
- `top_pod`

The runner enforces:

- a strict tool allowlist;
- context and namespace guards;
- blocked write and mutation verbs;
- spoofed tool-name detection;
- homoglyph and malformed-input rejection;
- schema validation;
- secret, token and PEM redaction;
- bounded action and fact budgets;
- terminal safety refusals.

Write operations such as `apply`, `patch`, `delete`, `exec`, `scale`,
`rollout restart`, `cordon` and `drain` are not permitted.

---

## Showing where every conclusion came from

KubeTriage does not only produce a diagnosis. It records a machine-checkable
chain showing how that diagnosis was reached.

In plain English:

```text
This conclusion
→ came from this fact
→ extracted from this exact part of the evidence
→ inside this evidence artifact
→ captured during this diagnostic run
```

The current provenance-aware path is:

```text
canonical redacted evidence artifact
→ exact extraction event
→ exact citation
→ typed fact
→ replayed classifier contribution
→ identity-bound support set
→ constrained claim
→ provenance-aware policy validation
→ immutable validated capability
→ deterministic renderer
```

This means the validator does not simply trust a submitted explanation.

It independently checks:

- that the evidence body matches its recorded identity;
- that the cited location exists;
- that byte ranges do not split UTF-8 characters;
- that the typed value really comes from the selected evidence;
- that the classifier contribution can be replayed from the verified facts;
- that the support set contains the correct contributing facts;
- that each claim is compatible with its evidence;
- that rejected output cannot reach the renderer.

This provides deterministic integrity checking. It is not a claim of
cryptographic authenticity, signed storage or tamper-proof infrastructure.

---

## How KubeTriage has been tested

The governed replay corpus contains:

| Corpus | Sessions | Purpose |
| --- | ---: | --- |
| Golden | 20 | Development regression |
| Holdout | 24 | Evaluation-only; never tuned against |
| Total | 44 | Frozen, governed replay sessions |

Current replay results:

| Check | Result |
| --- | ---: |
| Golden replay | **20/20 passed** |
| Holdout replay | **24/24 passed** |
| Base top-1 accuracy | **100%** |
| Brier score | **0.0557** |
| Expected Calibration Error | **0.2186** |
| False high-confidence results | **0** |
| Drift post-recheck accuracy | **100%** |
| Drift flip accuracy | **100%** |
| Drift confidence non-inflation violations | **0** |

The calibration corpus is small, so these figures should not be read as broad
production-performance claims. ECE remains above its target and is documented
rather than tuned away.

---

## Adversarial evaluation

KubeTriage includes a governed adversarial corpus designed to attack the
diagnostic and explanation boundary.

Examples include:

- swapping evidence between incidents;
- changing fact values;
- forging citations;
- replacing support sets;
- removing real contributors;
- inserting contributors from another incident class;
- changing drift state;
- rewriting identities and recalculating outer digests;
- attempting to render rejected output;
- crossing data between different runs.

The important cases do not rely on leaving an obviously stale checksum behind.
They recompute dependent identifiers and digests so that validation must reject
the attack for a semantic reason.

Current result:

| A5A adversarial corpus | Result |
| --- | ---: |
| Total governed cases | **102** |
| Passed | **102/102** |
| Unexpected admissions | **0** |
| Capability leaks | **0** |
| Renderer leaks | **0** |

Frozen corpus digest:

```text
60504978eaa046f9fe04b4c532ebf4c6039c74e2e42caa727312f384e370bb52
```

---

## Property and metamorphic evaluation

KubeTriage is also tested by transforming valid incidents and checking how the
result should relate to the original.

This is sometimes called metamorphic testing. In ordinary language, it asks
questions such as:

- Does reordering the evidence change the diagnosis?
- Can duplicated evidence make the system more confident?
- Can unrelated noise affect the winning class?
- Do operational IDs alter stable evidence identity?
- Does removing supporting evidence make the diagnosis stronger?
- Does the same incident produce identical output across repeated runs?

Current result:

| A5B property suite | Result |
| --- | ---: |
| Total governed properties | **67** |
| Passed | **67/67** |
| Capability leaks | **0** |
| Renderer leaks | **0** |
| Nondeterministic repeat failures | **0** |

Frozen suite digest:

```text
b367b02266444c0b680b28fb97ae60c197e4e00c2673aac3285fb82fcaebd124
```

### Evidence removal and confidence

A5B found an important distinction.

Removing only positive support must not increase confidence.

However, removing an evidence source may sometimes remove both:

- supporting information; and
- information that was reducing confidence.

That is a mixed-information change, so there is no honest universal rule saying
confidence must always decrease.

KubeTriage now records a confidence decomposition so the change can be explained
mechanically.

For the reviewed probe example:

```text
Before removal:
  confidence: 0.72
  evidence quality: mixed strong and weak
  mixed-evidence dampening: 0.16

After removing describe_pod:
  confidence: 0.88
  evidence quality: strong only
  mixed-evidence dampening: 0.00
```

The increase is fully explained by the removal of weak-evidence dampening.

This case is classified as mixed-information removal, not as proof that less
evidence should normally increase confidence.

Separate pure-support-removal tests cover all seven incident classes and require
confidence to remain equal or decrease.

---

## AI-agent readiness

KubeTriage itself is not an AI agent.

The project now has a strongly constrained boundary that a future explanation
layer could consume, including:

- a closed response envelope;
- immutable authority records;
- exact claim-to-evidence provenance;
- deterministic policy enforcement;
- contribution replay;
- support-set identity validation;
- adversarial testing;
- property and transformation testing;
- deterministic rendering.

That still does not mean a real agent should be connected.

Real-agent admission remains blocked because operational questions still need to
be addressed, including:

- provider and model isolation;
- authentication and authorisation;
- tenant separation;
- rate and concurrency limits;
- request lifecycle and idempotency;
- audit retention;
- trace correlation;
- revocation and kill-switch procedures;
- human approval;
- operational failure handling;
- shadow-mode evaluation.

There is currently:

- no real LLM provider integration;
- no autonomous AI agent;
- no cluster credentials exposed to a model;
- no live production diagnosis mode;
- no remediation;
- no kubectl mutation path.

---

## Current project status

| Gate | Status |
| --- | --- |
| Deterministic diagnostic engine | **Complete** |
| Replay harness | **Complete** |
| Golden and holdout corpora | **Complete** |
| Calibration evaluation | **Complete** |
| Strict safe-response envelope | **Complete** |
| Constrained explanation policy | **Complete** |
| Immutable run manifest and stable evidence identity | **Passed independent review** |
| Exact claim → fact → evidence provenance | **Passed independent review through H4** |
| Governed adversarial admission corpus | **Passed independent review — 102/102** |
| Property and metamorphic evaluation | **Passed independent review — 67/67** |
| A5 adversarial evaluation programme | **Complete** |
| Real-agent admission | **Blocked** |
| Remediation or cluster mutation | **Not implemented** |

---

## Sample outputs

| Sample | What it demonstrates |
| --- | --- |
| [Image-pull diagnosis](sample-output/engine-imagepull-demo.txt) | Normal deterministic diagnosis |
| [Drift class flip](sample-output/engine-drift-demo.txt) | Full drift recomputation and confidence non-inflation |
| [Safety refusal](sample-output/engine-refusal-demo.txt) | Terminal refusal for an unsafe request |
| [Safe response envelope](sample-output/agent-safe-response-demo.json) | Earlier closed `agent_safe_response/v1` authority envelope |
| [Deterministic consumer explanation](sample-output/agent-consumer-demo.txt) | Offline explanation scaffold; not a real AI agent |
| [LLM policy rejection](sample-output/llm-policy-demo.txt) | Offline legacy policy demonstration; no real LLM is called |

The safe-response and LLM-policy examples represent earlier boundary milestones.
The private implementation now also contains the later provenance-aware v3
admission path described above.

---

## Demo flow

The main demonstration uses three frozen incidents:

### 1. Normal diagnosis

An image-pull failure is classified as `imagepull` with confidence `0.88`.

This demonstrates:

- deterministic diagnosis;
- hypothesis ranking;
- read-only operation;
- reproducible reporting.

### 2. Drift recomputation

Initial evidence supports `crashloop`.

New evidence introduces stronger image-pull signals, so KubeTriage recomputes
the full incident and changes the top class to `imagepull`.

This demonstrates:

- one bounded drift recheck;
- complete recomputation;
- class change when evidence changes;
- confidence non-inflation;
- preserved provenance.

### 3. Safety refusal

A request targeting an unsafe cluster context is refused.

This demonstrates:

- safety enforced in code;
- refusal as a successful terminal outcome;
- no diagnostic continuation;
- no remediation attempt.

See [docs/demo-walkthrough.md](docs/demo-walkthrough.md).

---

## Optional local fixture lab

The implementation repository includes an optional local `kind` lab for
creating controlled broken workloads and capturing read-only evidence.

The flow is:

```text
local kind cluster
→ controlled broken workload
→ read-only evidence capture
→ generated replay session
→ human review and redaction
→ governed corpus promotion
→ offline replay diagnosis
```

Live capture is not the diagnostic authority.

Generated sessions are unreviewed and are not evaluation material until they
have been inspected and promoted under corpus governance.

No live Kubernetes cluster is required for the normal demo, tests or evaluation.

---

## Honest limitations

KubeTriage currently has important limits:

- it supports seven pattern-based incident classes;
- it does not cover every Kubernetes failure mode;
- the replay corpus is governed but small;
- ECE remains above its target;
- confidence uses a small number of deterministic tiers;
- arbitrary evidence removal does not have a universal confidence direction;
- deterministic integrity is not cryptographic authenticity;
- the implementation repository remains private;
- there is no real provider or model integration;
- there is no production live-diagnosis service;
- there is no remediation or mutation path;
- real-agent admission remains blocked.

The system returns `insufficient_evidence` rather than guessing when the
available evidence does not justify a supported diagnosis.

---

## Repository note

This showcase repository documents:

- the project’s purpose;
- the architecture;
- the safety model;
- the current evaluation status;
- representative outputs.

The full implementation repository remains private while the project is under
active development.

The showcase does not include:

- implementation source code;
- private test suites;
- governed replay corpora;
- raw Kubernetes evidence;
- cluster credentials;
- provider integration.

---

## Documentation

| Document | Purpose |
| --- | --- |
| [Architecture](docs/architecture.md) | Data flow, components and invariants |
| [Safety model](docs/safety-model.md) | Read-only boundaries, refusals and governance |
| [Demo walkthrough](docs/demo-walkthrough.md) | How to present KubeTriage in 3–10 minutes |
| [Live fixture pipeline](docs/live-fixture-pipeline.md) | Optional local kind capture workflow |

---

## In one sentence

> KubeTriage is a read-only Kubernetes incident investigator that works from
> frozen evidence, produces the same diagnosis every time, and can show exactly
> where every conclusion came from.
