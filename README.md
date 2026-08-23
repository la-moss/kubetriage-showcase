# KubeTriage

**A deterministic evidence engine for bounded Kubernetes incident diagnosis — built to answer a harder question than "what's wrong with this pod": can a diagnostic system produce conclusions whose evidence, confidence, provenance, and evaluation history are reproducible and independently inspectable?**

KubeTriage turns governed, frozen operational evidence into replayable, evidence-backed diagnoses. The same evidence always produces the same result. Every admitted conclusion links to the exact evidence locations that support it. When the evidence doesn't justify a conclusion, the engine can abstain rather than guess.

Diagnostic authority is deterministic code. Safety is enforced in code, not prompts.

---

## The problem it addresses

Production incidents rarely fail on a single signal. Diagnosis means correlating workload state, configuration, events, and logs before you can defend a conclusion — and that reasoning can be difficult to replay once the incident has moved on.

Existing Kubernetes diagnostic tools already detect common failure patterns well, typically with rule-based analyzers operating directly against live clusters, sometimes with optional AI-generated explanation such as K8sGPT.

KubeTriage is not trying to be a broader analyzer.

It investigates a different property:

> **Can a bounded diagnostic system make its conclusions auditable?**

For KubeTriage, that means being able to trace a conclusion through:

```text
claim
→ classifier / hypothesis contribution
→ typed fact
→ exact location in frozen evidence
→ governed evidence artefact
→ sealed run identity
```

Confidence is explicit and its calibration is evaluated separately from correctness. Abstention is a valid outcome. The evidence that produced a diagnosis can be replayed later rather than relying on whatever state the cluster happens to have at that moment.

That property becomes particularly relevant where diagnostic conclusions need to survive later scrutiny — for example during post-incident review or in environments with strong audit requirements.

---

## What it does today

On governed Kubernetes replay evidence, KubeTriage:

- uses evidence produced through a strict **six-tool read-only allowlist** (`events_tail`, `describe_pod`, `describe_deploy`, `logs`, `get_yaml`, `top_pod`);
- refuses write or mutation capabilities in code;
- validates, redacts, and canonicalises evidence before governed diagnostic artefacts are admitted;
- extracts **typed facts with exact citations**;
- classifies against **seven bounded incident classes**;
- ranks hypotheses and computes confidence from explicit evidence inputs;
- measures diagnostic correctness and confidence behaviour as separate claims;
- performs **at most one bounded drift recheck** with full recomputation rather than patching the previous answer;
- keeps confidence changes attributable to an explicit decomposition, with governed non-inflation invariants where applicable;
- seals an **immutable run manifest** with claim → fact → evidence provenance;
- returns `insufficient_evidence` when evidence within the governed diagnostic domain does not justify a supported conclusion;
- returns terminal `refusal` when a request violates a safety boundary.

### Seven supported classes

`crashloop` · `imagepull` · `oom` · `pending` · `service_unreachable` · `probe_failure` · `config_dependency_failure`

These are seven bounded Kubernetes failure patterns, not Kubernetes generally.

Within that boundary, KubeTriage can return `insufficient_evidence` when the available evidence does not justify a diagnosis. Incidents outside the governed diagnostic domain are treated separately rather than forced into a supported class.

---

## How it works

```text
frozen read-only evidence
  → validation and redaction
  → typed facts with exact citations
  → deterministic classification
  → hypothesis ranking
  → confidence decomposition
  → optional bounded drift recheck
  → immutable run manifest
  → claim / fact / evidence provenance
  → provenance-aware policy validation
  → deterministic operator-visible output
```

Replay is the diagnostic source of truth: no hidden diagnostic state, no online learning, and no randomness in diagnosis.

### Delivery path today

Governed read-only capture exists today for local `kind` fixtures under `labs/kind/`.

Capture and diagnosis are deliberately separate:

```text
local kind cluster
→ bounded read-only capture
→ frozen evidence
→ deterministic diagnosis
```

The diagnostic engine consumes recorded, replayable evidence rather than reasoning directly against continuously changing live cluster state.

The intended future operational path is:

```text
real cluster
→ bounded read-only capture
→ validate / redact / freeze
→ deterministic diagnosis
→ provenance report
```

General real-cluster capture is a **post-evaluation roadmap item**, not a current capability.

Replayability remains the diagnostic authority model even if evidence capture later becomes operational.

See [Architecture](docs/architecture.md) · [Safety model](docs/safety-model.md) · [Live fixture pipeline](docs/live-fixture-pipeline.md).

---

## Evaluation

KubeTriage separates controlled evaluation from independently occurring real incidents.

That distinction is fundamental to the project.

### Tier 1 — controlled corpora

Golden, holdout, adversarial, and metamorphic suites use controlled, author-constructed evaluation material.

| Suite | Reviewed result |
|---|---:|
| Golden | 20/20 |
| Controlled holdout | 24/24 |
| Adversarial | 102/102 |
| Metamorphic | 67/67 |

For the controlled non-drift diagnostic holdout:

| Metric | Result |
|---|---:|
| Top-1 accuracy | 100% |
| Brier score | 0.0557 |
| ECE | 0.2186 |
| False high confidence | 0 |

ECE is above the project's `0.15` controlled-corpus target.

It is reported rather than tuned away.

For the controlled drift set:

| Metric | Result |
|---|---:|
| Post-drift Top-1 | 100% |
| Drift flip accuracy | 100% |
| Confidence non-inflation violations | 0 |

These results establish behaviour on material constructed and governed by the project.

**They do not establish real-world diagnostic reliability.**

KubeTriage was not trained on these corpora, and the controlled results are not presented as production accuracy claims.

### Tier 2 — independently occurring real incidents

This is the evaluation that matters most for the current programme.

KubeTriage is acquiring independently occurring Kubernetes incidents under a frozen, support-blind acquisition protocol.

Current governed state:

```text
registered candidates: 4
admitted incident families: 2
acquisition: OPEN
V2 blind evaluation: NOT STARTED
```

The diagnostic engine remains frozen at:

```text
v2.8.0-a6b-pre-s4-result-inventory-reviewed-baseline
```

Commit:

```text
d3a1b85e5232a36192ed55aad9f84db090e88cc4
```

Nothing learned from real-incident acquisition is permitted to tune that frozen engine before the governed evaluation.

#### Support-blind acquisition

Real incidents are discovered under:

```text
V1-RIC-ACQ-PROTOCOL-L1-001
```

Acquisition decisions must not depend on:

- whether KubeTriage supports the failure mode;
- whether the incident is expected to be easy or difficult;
- whether including it would improve evaluation results;
- what KubeTriage would diagnose;
- what confidence KubeTriage would assign.

The corpus is therefore not intentionally selected around the engine's current strengths.

#### Staged separation

The real-incident programme separates:

```text
discovery
→ human admission
→ ground-truth adjudication
→ representability / coverage analysis
→ governed evaluation
```

These are different authorities.

Admission is not ground truth.

Ground truth is not representability.

Representability is not determined from KubeTriage output.

#### Blinded ground truth

The evaluation design requires per-family ground truth to be frozen before incident-specific KubeTriage output for that family may be generated or viewed.

That includes:

- diagnosis;
- confidence;
- support status;
- hypotheses;
- `insufficient_evidence`;
- refusal;
- other incident-specific diagnostic interpretation.

The mechanical GT and blinding authority has been implemented and independently reviewed.

**Real ground-truth adjudication has not begun.**

#### Pre-committed analysis

Before V2 execution begins, the analysis plan was frozen prospectively.

It will define, before results are revealed:

- evaluation unit;
- evaluation cohort;
- correctness denominator;
- broader end-to-end denominator;
- coverage reporting;
- abstention handling;
- unresolved-ground-truth handling;
- exclusions and withdrawals;
- calibration rules, if retained;
- scoring implementation;
- reporting requirements;
- permissible claims given the eventual corpus size.

The V2 analysis plan is **frozen** as evaluation methodology. V2 diagnostic execution has **not** started, and no V2 scoring result currently exists.

#### Honest stopping

The acquisition protocol targets at least 30 admitted incident families and has an independent hard stop:

```text
2026-11-09T23:59:59Z
```

Thirty incidents are not guaranteed.

If the hard stop is reached first, acquisition closes at the corpus size actually obtained.

The project does not extend acquisition simply to manufacture the desired denominator.

Any limitation caused by corpus size will be reported directly.

Current corpus state is tracked in [docs/current-status.md](docs/current-status.md).

---

## Coverage may matter more than accuracy

A system can achieve high accuracy inside a narrow domain while covering only a small part of the incidents that actually occur.

Those are different claims.

KubeTriage's real-incident evaluation is therefore intended to answer both:

```text
When KubeTriage has diagnostic authority,
how often is it correct?
```

and:

```text
Across independently occurring incidents,
how often does that bounded authority actually apply?
```

If the seven current classes cover only a fraction of independently acquired incidents, that may be more informative than a headline accuracy percentage.

The purpose of the evaluation is to expose that boundary rather than hide it.

---

## Safety and boundaries

| Property | Enforcement |
|---|---|
| **Read-only** | Six-tool capability boundary; mutation and write capabilities refused |
| **No remediation** | No apply, patch, delete, restart, scale, or automated corrective action |
| **Deterministic diagnosis** | Same governed evidence produces the same diagnostic result |
| **Bounded authority** | Seven explicit diagnostic classes with separate abstention and refusal behaviour |
| **Diagnosis ≠ explanation** | A future explanation layer may describe an admitted result but cannot silently replace diagnostic authority |
| **No autonomous agent in diagnosis** | No provider or model currently holds diagnostic authority |
| **Human-controlled action** | KubeTriage reports; humans decide what operational action to take |

Safety is enforced through code and governed capability boundaries rather than prompt instructions.

Earlier project phases built and tested agent-admission policy, output contracts, provenance-aware explanation boundaries, and offline scaffolding to establish where authority boundaries would sit if an explanation agent were ever admitted.

No provider-driven autonomous diagnostic agent ever operated as part of KubeTriage.

The useful result of that work was the authority boundary:

> Where deterministic diagnostic authority exists, a future probabilistic explanation layer may explain the admitted result but must not silently override it.

---

## Try the samples

Representative public outputs require no cluster:

- [Image-pull diagnosis](sample-output/engine-imagepull-demo.txt) — normal deterministic diagnosis
- [Drift class flip](sample-output/engine-drift-demo.txt) — full recomputation when new evidence changes the leading diagnosis
- [Safety refusal](sample-output/engine-refusal-demo.txt) — terminal refusal for an unsafe cluster context
- [Provenance-aware explanation](sample-output/provenance-aware-explanation-demo.json) — evidence-backed explanation and provenance structure
- [Confidence decomposition](sample-output/confidence-decomposition-demo.json) — explicit confidence-component behaviour
- [Adversarial evaluation summary](sample-output/adversarial-evaluation-summary.json)
- [Metamorphic evaluation summary](sample-output/metamorphic-evaluation-summary.json)

Presenter walkthrough: [docs/demo-walkthrough.md](docs/demo-walkthrough.md).

---

## Limitations

KubeTriage deliberately makes its current limitations visible.

- **Seven pattern-based diagnostic classes.** It does not provide broad Kubernetes or cloud-native diagnosis.
- **Small controlled evaluation corpus.** Tier 1 results are not production-reliability claims.
- **Real-world diagnostic reliability is not yet established.** V2 has not run.
- **Real-incident coverage is unknown.** The support-blind corpus is being collected specifically so this can be measured rather than assumed.
- **General real-cluster capture is not implemented.** Current governed capture is limited to local `kind` fixtures.
- **No remediation.** KubeTriage cannot repair a cluster.
- **Single maintainer.** Real-incident adjudication does not yet have independent multi-rater evidence.
- **The main implementation repository is currently private.** The public showcase therefore cannot make every implementation claim independently reproducible today.
- **Evidence integrity is not evidence authenticity.** Deterministic digests can show that governed artefacts match their recorded identity; they do not cryptographically prove that captured cluster evidence was originally truthful.
- **External witnessing is not claimed where it does not yet exist.** Project-recorded hashes identify frozen artefacts, but a hash alone is not independent proof of when those bytes existed.

These are constraints on what can currently be claimed, not details hidden behind the evaluation numbers.

---

## Verifying these claims

### Frozen diagnostic baseline

```text
tag:
v2.8.0-a6b-pre-s4-result-inventory-reviewed-baseline

commit:
d3a1b85e5232a36192ed55aad9f84db090e88cc4
```

### Real-incident acquisition protocol

```text
protocol:
V1-RIC-ACQ-PROTOCOL-L1-001

active version:
1.1.0

SHA-256:
e34b45467b60f71413f621c0796f301ea902840788cb63f7243499fdc51f1e5c
```

The digest identifies the project-recorded frozen protocol bytes.

No independent external timestamp is claimed here.

### V2 analysis authority

```text
status:
FROZEN (methodology)

V2 diagnostic execution:
NOT STARTED
```

A frozen analysis plan is not an executed evaluation. No V2 scores or reveal exist.

### Public evidence

- [Current status](docs/current-status.md)
- [Architecture](docs/architecture.md)
- [Safety model](docs/safety-model.md)
- [Live fixture pipeline](docs/live-fixture-pipeline.md)
- [Demo walkthrough](docs/demo-walkthrough.md)
- [Sample outputs](sample-output/)

Where a claim cannot currently be independently verified because the implementation or protected evaluation material is private, that limitation is stated rather than elided.

---

## Current programme

```text
frozen v2.8 diagnostic engine
        ↓
support-blind real-incident acquisition
        ↓
human admission
        ↓
ground-truth / blinding authority
        ↓
representability / coverage authority
        ↓
pre-committed V2 analysis + scoring authority
        ↓
blind real-incident evaluation
        ↓
evidence-driven revision, if justified
```

The current priority is **evidence**, not adding diagnostic capability.

The engine remains frozen while the project determines how it performs against incidents that were not constructed around it.

Only after that evaluation will broader diagnostic expansion or general real-cluster productisation be reconsidered.

---

## Documentation

- [Architecture](docs/architecture.md)
- [Safety model](docs/safety-model.md)
- [Live fixture pipeline](docs/live-fixture-pipeline.md)
- [Current status](docs/current-status.md)
- [Demo walkthrough](docs/demo-walkthrough.md)
- [Sample outputs](sample-output/)

---

*KubeTriage is an engineering research project by [L. A. Moss](https://github.com/la-moss) exploring what auditable diagnostic systems require: bounded authority, provenance, abstention, replay, and evaluation methods capable of distinguishing genuine evidence from evidence shaped around the system being tested.*
