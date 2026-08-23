# KubeTriage

**Deterministic, evidence-grounded Kubernetes incident diagnosis.**

KubeTriage turns frozen, read-only Kubernetes evidence into structured facts,
diagnoses supported failure patterns, ranks hypotheses, calculates explicit
confidence, and produces reproducible reports with exact claim → fact → evidence
provenance.

The same evidence produces the same diagnosis. Every admitted conclusion can be
traced through the reasoning that produced it to the evidence that supported it.

KubeTriage currently diagnoses seven bounded Kubernetes incident classes and is
being evaluated through controlled replay, adversarial and metamorphic testing,
and a prospective blind real-incident validation programme.

The implementation lives in the private repo

## Current status

Frozen diagnostic baseline:
[`v2.8.0-a6b-pre-s4-result-inventory-reviewed-baseline`](https://github.com/la-moss/KubeTriage/tree/v2.8.0-a6b-pre-s4-result-inventory-reviewed-baseline)
(`d3a1b85e5232a36192ed55aad9f84db090e88cc4`)

| Area | Status |
| --- | --- |
| Deterministic diagnostic engine (7 classes) | Implemented / frozen at v2.8 |
| Controlled replay evaluation | Golden 20/20, holdout 24/24 |
| Adversarial / metamorphic evaluation | 102/102 and 67/67 |
| Exact claim → fact → evidence provenance | Implemented |
| Support-blind real-incident acquisition | Open |
| V2 analysis methodology | Frozen |
| V2 diagnostic execution | Not started |

## How it works

KubeTriage diagnoses a frozen evidence bundle through a deterministic pipeline:

```text
read-only evidence
  → validation and redaction
  → typed facts with exact citations
  → deterministic classification
  → hypothesis ranking
  → confidence decomposition
  → optional bounded drift recheck
  → immutable run identity
  → claim → fact → evidence provenance
  → diagnostic report
```

**Evidence intake** accepts only the six-tool allowlist (`events_tail`,
`describe_pod`, `describe_deploy`, `logs`, `get_yaml`, `top_pod`). Validation
rejects spoofing, homoglyphs, injection, and schema deviation; redaction removes
secrets, JWTs, and PEM material before diagnosis.

**Fact extraction** turns those artefacts into typed facts with extraction-time
citations (JSON Pointer or UTF-8 byte range). Citations are captured when the
fact is extracted, not reconstructed later by searching the report text.

**Classification and ranking** score the seven supported classes, emit a
winning class when the evidence justifies one, and rank remaining hypotheses.

**Confidence** is calculated from explicit evidence inputs and decomposed into
inspectable components. Accuracy and calibration are measured separately.

**Drift recheck**, when new governed evidence arrives, runs once with full
recomputation. The previous answer is not patched, and confidence does not
inflate after drift.

**Sealing** records an immutable run identity, then binds every admitted claim
back through classifier or hypothesis contributions to typed facts, exact
evidence locations, and the sealed run.

The pipeline is a state machine:
`SIGNAL_INGESTION → FACT_EXTRACTION → CLASSIFICATION → HYPOTHESIS_RANKING → CONFIDENCE_SCORING → STOP_OR_NEXT → FINAL_REPORT`.
A drift recheck re-enters at `FACT_EXTRACTION` at most once.

Diagnostic authority lives in deterministic code. The same evidence, versions,
and policy produce the same diagnosis, facts, confidence, and provenance.

## Supported diagnosis

| Class | Description |
| --- | --- |
| `crashloop` | Container repeatedly fails and restarts |
| `imagepull` | Image cannot be pulled from the registry |
| `pending` | Pod cannot be scheduled |
| `service_unreachable` | Service has no reachable endpoints |
| `oom` | Container killed by the OOM killer |
| `probe_failure` | Readiness / liveness probe failure (grouped) |
| `config_dependency_failure` | Missing or invalid ConfigMap or Secret dependency (grouped) |

`insufficient_evidence` is a valid terminal outcome when governed evidence does
not justify a supported diagnosis. Requests that violate a safety boundary
terminate through refusal: the diagnostic loop does not continue.

Coverage is pattern-based over these classes. Broader Kubernetes and
cloud-native diagnosis remains future work.

## Evidence and provenance

Every admitted conclusion is bound to governed evidence:

```text
claim
  → classifier / hypothesis contribution
  → typed fact
  → exact JSON Pointer or UTF-8 byte range
  → governed redacted evidence artefact
  → sealed run identity
```

Key properties:

- immutable run manifest (`kubetriage_run_manifest/v1`) with deterministic
  digests and a diagnostic fingerprint
- stable evidence-artefact identity over canonical redacted bodies
- stable typed-fact identity with extraction-time citations
- contribution replay from the verified fact graph
- support-set identity validation by independent recomputation
- provenance-aware policy v3, which mints an immutable validated capability
  before the deterministic renderer runs

See the implementation repository for the corresponding source and full technical documentation.

## Replay and determinism

KubeTriage diagnoses frozen evidence so an investigation can be rerun against
the same inputs that produced the original conclusion.

| Corpus | Sessions | Role |
| --- | ---: | --- |
| Golden | 20 | Development regression |
| Controlled holdout | 24 | Controlled evaluation only; never tuned against |

Same input yields the same result. One bounded drift recheck is permitted, and
it recomputes the diagnosis in full. Replay is the inspection path for later
review of a conclusion.

## Confidence

Confidence is calculated from explicit evidence inputs. Diagnostic correctness
and calibration are different claims.

**Base (non-drift, 14 diagnostic sessions):**

| Metric | Value |
| --- | --- |
| Top-1 accuracy | 100% |
| Brier score | 0.0557 |
| ECE | 0.2186 *(small controlled corpus; above 0.15 target)* |
| False high confidence (≥ 0.80) | 0 |

**Drift (6 sessions):**

| Metric | Value |
| --- | --- |
| Top-1 accuracy post-drift | 100% |
| Drift flip accuracy | 100% |
| Confidence non-inflation violations | 0 |

Pure-support removal and mixed-information removal are distinct:

| Relation | Meaning |
| --- | --- |
| **Pure support removal** | Only winning-class supporting evidence is removed → confidence must not increase |
| **Mixed-information removal** | Support and confidence-dampening inputs change together → no universal confidence direction; the decomposition must explain the change |

## Evaluation

### Controlled evaluation

| Suite | Reviewed result |
| --- | ---: |
| Golden | 20/20 |
| Controlled holdout | 24/24 |
| Adversarial | 102/102 |
| Metamorphic | 67/67 |

These results describe behaviour on governed controlled corpora. Real-incident
performance is being evaluated separately.

### Real-incident validation

KubeTriage is now in its real-incident validation programme.

The diagnostic baseline is frozen. The blind V2 analysis methodology is also
frozen before any results are produced. Support-blind acquisition remains open
while ground truth, representability, and the evaluation cohort have not yet
been created.

Freezing the diagnostic system and evaluation method before results are produced
reduces the opportunity to change either in response to the eventual outcome.

| Stage | State |
| --- | --- |
| Diagnostic baseline | Frozen |
| Real-incident acquisition | Open |
| V2 methodology | Frozen |
| Ground truth | Not started |
| Representability | Not started |
| Evaluation cohort | Not created |
| V2 execution | Not started |

Frozen V2 methodology identity: 35136 bytes, SHA-256
`9a1f15dc0347d0bc7aecf0bf082e5029ea23e1426fd0405baf4e4a483ba5fb67`.
Earliest Traversal-2 eligibility is `2026-08-31T00:00:00Z`.

## Architecture

| Layer | Responsibility |
| --- | --- |
| Evidence | Validate, redact, and freeze diagnostic inputs |
| Fact | Extract typed facts with exact citations |
| Diagnostic | Classify, rank hypotheses, and calculate confidence |
| Provenance | Bind conclusions back to facts and evidence |
| Policy | Control which results may be admitted and rendered |
| Evaluation | Replay, adversarial, metamorphic, and real-incident validation |

Where a deterministic evaluator supports a domain, a later probabilistic layer
must not silently override its admitted result. Outside those domains, any
later conclusion would need to be represented as a hypothesis rather than
diagnostic authority.

## Design boundaries

KubeTriage currently operates as a read-only diagnostic authority over seven
bounded Kubernetes incident classes. Cluster mutation and remediation sit
outside that authority. Probabilistic components may later assist investigation,
interaction, or explanation; the deterministic diagnostic layer remains the
authority within the domains it supports. Broader Kubernetes and cloud-native
coverage remains future work.

## Try the samples

Representative public outputs require no cluster:

- [Image-pull diagnosis](sample-output/engine-imagepull-demo.txt)
- [Drift class flip](sample-output/engine-drift-demo.txt)
- [Safety refusal](sample-output/engine-refusal-demo.txt)
- [Provenance-aware explanation](sample-output/provenance-aware-explanation-demo.json)
- [Confidence decomposition](sample-output/confidence-decomposition-demo.json)
- [Adversarial evaluation summary](sample-output/adversarial-evaluation-summary.json)
- [Metamorphic evaluation summary](sample-output/metamorphic-evaluation-summary.json)

## For technical reviewers

The actual implementation, tests and specifications are in the public
[KubeTriage repository](https://github.com/la-moss/KubeTriage).

Start with the
[technical review guide](https://github.com/la-moss/KubeTriage/blob/main/docs/technical-review-guide.md).
It maps the review surface across the state machine, classifier, confidence,
provenance, safety, replay harness, adversarial and metamorphic tests, and the
constitution.

Useful questions include whether diagnostic authority is actually deterministic
in code, whether the read-only boundary is enforceable, whether provenance can
be substituted after a run, whether replay reproduces the decision, whether
confidence is defensible from its decomposition, and whether any boundary is
unnecessarily restrictive.

## Current status and documentation

- [Current status](docs/current-status.md)
- [Architecture](docs/architecture.md)
- [Safety model](docs/safety-model.md)
- [Live fixture pipeline](docs/live-fixture-pipeline.md)
- [Demo walkthrough](docs/demo-walkthrough.md)
- [Sample outputs](sample-output/)

## Known limitations

- Seven pattern-based incident classes
- Kubernetes-native governed evidence today
- ECE remains above target on the current controlled holdout set
- Evaluation covers governed known attacks and declared transformations, not
  formal verification
- No cryptographic signing or authenticity layer

---

*KubeTriage is an engineering research project by [L. A. Moss](https://github.com/la-moss) exploring bounded diagnostic authority, provenance, abstention, replay, and evaluation methods for evidence-grounded Kubernetes diagnosis.*
