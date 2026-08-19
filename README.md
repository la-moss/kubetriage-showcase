# KubeTriage

**A deterministic evidence engine for bounded Kubernetes incident diagnosis — built to answer a harder question than "what's wrong with this pod": can a diagnostic system produce conclusions whose evidence, confidence, provenance, and evaluation history are reproducible and independently inspectable?**

KubeTriage turns governed, frozen operational evidence into replayable, evidence-backed diagnoses. The same evidence always produces the same result. Every conclusion links to the exact evidence locations that support it. When the evidence doesn't justify a conclusion, the engine says so.

Diagnostic authority is deterministic code. Safety is enforced in code, not prompts.

---

## The problem it addresses

Production incidents rarely fail on a single signal. Diagnosis means correlating workload state, configuration, events, and logs before you can defend a conclusion — and that reasoning is slow to replay and easy to dispute afterwards.

Existing Kubernetes diagnostic tools already detect common failure patterns well, typically with rule-based analyzers operating directly against live clusters, sometimes with optional AI-generated explanation (e.g., K8sGPT). KubeTriage is not trying to be a broader analyzer. It investigates a different property: **whether a narrow diagnostic system can make every conclusion auditable** — traceable through claim → typed fact → exact location in frozen evidence, with calibrated confidence, explicit abstention, and a frozen evaluation record.

That property matters most exactly where explanations are most contested: regulated environments, post-incident review, and any setting where "the tool said so" is not an acceptable answer.

## What it does today

On governed Kubernetes replay evidence, KubeTriage:

- collects evidence through a strict **six-tool read-only allowlist** (`events_tail`, `describe_pod`, `describe_deploy`, `logs`, `get_yaml`, `top_pod`) — write verbs are refused in code;
- validates, redacts, and canonicalises evidence before any diagnostic artefact is admitted;
- extracts **typed facts with exact citations**;
- classifies against **seven bounded incident classes** (below);
- ranks hypotheses and computes confidence from explicit evidence inputs — accuracy and calibration are measured as separate claims;
- performs **at most one bounded drift recheck** with full recomputation — confidence cannot inflate silently when evidence changes;
- seals an **immutable run manifest** with complete claim → fact → evidence provenance;
- returns `insufficient_evidence` rather than guessing, and `refusal` — terminal, no further processing — on safety violations.

**Seven supported classes:** crashloop · imagepull · oom · pending · service_unreachable · probe_failure · config_dependency_failure

These are bounded pattern-based coverage — the most common Kubernetes failure modes, not Kubernetes generally. Everything outside them is an honest `insufficient_evidence`.

## How it works

```
frozen read-only evidence
  → validation and redaction
  → typed facts with exact citations
  → classification → hypothesis ranking → confidence
  → optional one bounded drift recheck (full recomputation)
  → run manifest + provenance bundle
  → provenance-aware policy validation
  → deterministic operator-visible output
```

Replay is the diagnostic source of truth: no hidden state, no online learning, no randomness.

**Delivery path today:** governed read-only capture exists for local kind fixtures (`labs/kind/`, context-guarded); the diagnostic engine consumes the frozen, replayable evidence that capture produces — never live mutable cluster state. The intended operational path is:

```
real cluster → bounded read-only capture → validate/redact/freeze → deterministic diagnosis → provenance report
```

General real-cluster capture is a post-evaluation roadmap item, not current capability. See [Architecture](docs/architecture.md) · [Safety model](docs/safety-model.md) · [Live fixture pipeline](docs/live-fixture-pipeline.md).

## Evaluation

KubeTriage is being evaluated in two tiers, and the distinction between them is the point.

### Tier 1 — controlled corpora (complete)

Golden, holdout, adversarial, and metamorphic suites, all synthetic and author-constructed:

| Suite | Result |
|---|---|
| Golden | 20/20 |
| Controlled holdout | 24/24 (14 non-drift sessions: top-1 100%, Brier 0.0557, 0 false high-confidence) |
| Adversarial | 102/102, zero unexpected admissions |
| Metamorphic | 67/67 |

These establish that the engine behaves correctly on material constructed around it. ECE on the holdout is 0.2186 — above the 0.15 target on this small set, reported rather than tuned away. **None of this establishes real-world diagnostic reliability**, and it is not presented as doing so.

### Tier 2 — independently occurring real incidents (in progress)

The evaluation that actually matters is being built under governance designed to keep it honest:

- **Support-blind acquisition**: a frozen, versioned acquisition protocol (`V1-RIC-ACQ-PROTOCOL-L1-001`) governs how real incidents are discovered. Acquisition decisions are blind to whether KubeTriage supports the failure mode — the corpus cannot quietly become a dataset selected around the engine's strengths.
- **Staged separation**: discovery, human admission, ground-truth adjudication, and representability analysis are separate decisions. Admission does not depend on engine capability.
- **Blinded ground truth**: per-family ground truth is frozen before any engine output for that family is generated or viewed.
- **Pre-committed analysis**: the V2 analysis plan — endpoints, denominators, calibration rules, and what claims are permissible at whatever corpus size the hard stop produces — is frozen before evaluation begins.
- **Honest stopping**: acquisition targets ≥30 admitted incident families with an independent hard stop of 2026-11-09. If the hard stop arrives first, the corpus closes at whatever size it reached, and that limitation is reported, not worked around.

Corpus state counters are published at [docs/current-status.md](docs/current-status.md); acquisition is currently open. The diagnostic engine remains frozen at `v2.8.0-a6b-pre-s4-result-inventory-reviewed-baseline` throughout — nothing learned from real incidents touches it until the blind evaluation is complete.

The expected result may be about **coverage as much as accuracy**: if the seven classes cover only a fraction of independently occurring incidents, that is a more informative finding than a headline percentage, and the analysis plan is built to report it.

## Safety and boundaries

| Property | Enforcement |
|---|---|
| Read-only | Six-tool allowlist; write verbs refused in code |
| No remediation | No apply, patch, delete, restart, scale — humans decide and perform fixes |
| Diagnosis ≠ explanation | Diagnosis is deterministic; any future explanation layer may describe an admitted result but cannot change it |
| Bounded authority | Seven classes; `insufficient_evidence` and terminal `refusal` are first-class outcomes |
| No autonomous agent | No provider or model is admitted into the diagnostic path |

Earlier project phases (A4–A6) built and tested agent-admission policy, output contracts, and offline scaffolds to establish where authority boundaries would sit if an explanation agent were ever admitted. No provider-driven agent loop ever operated; that work is documented in [project history](docs/project-history.md).

## Try the samples

Representative outputs, no cluster required:

- [Image-pull diagnosis](samples/imagepull.md) — normal deterministic diagnosis (confidence 0.88)
- [Drift class flip](samples/drift-flip.md) — full recomputation under new evidence; confidence does not inflate
- [Safety refusal](samples/refusal.md) — terminal refusal for unsafe cluster context
- [Provenance bundle](samples/provenance.md) — claim → fact → evidence chain end to end
- [Confidence decomposition](samples/confidence.md) — mixed-information removal explained mechanically (0.72 → 0.88)

Presenter script: [docs/demo-walkthrough.md](docs/demo-walkthrough.md).

## Limitations

- Seven pattern-based classes — not broad Kubernetes or cloud-native coverage.
- Small controlled evaluation corpus; Tier 1 numbers are not production claims.
- Real-world diagnostic reliability is **not yet established** — the Tier 2 evaluation has not run.
- Replay artefacts carry deterministic integrity checking; governance artefacts (protocol freezes) carry cryptographic hashes with external timestamps. Neither is a claim of cryptographic authenticity of cluster evidence itself.
- Single maintainer; no independent multi-rater adjudication yet.
- The main implementation repository is currently private; this showcase carries the verification pack below so that public claims remain inspectable.

## Verifying these claims

- Frozen engine baseline digest: `[baseline digest]` — recorded at `[external timestamp location]`
- Acquisition protocol v1.1.0 hash: `e34b4546…51f1e5c` — recorded at `[location]`
- V2 analysis plan hash: `[to be added at Stage 1 freeze]`
- Corpus state counters (counts only, never contents): [docs/current-status.md](docs/current-status.md)
- Sample provenance bundles: [samples/](samples/)

Where an artefact cannot be published while the implementation is private, the limitation is stated rather than elided.

## Docs

[Architecture](docs/architecture.md) · [Safety model](docs/safety-model.md) · [Live fixture pipeline](docs/live-fixture-pipeline.md) · [Evaluation methodology](docs/evaluation-methodology.md) · [Project history](docs/project-history.md) · [Current status](docs/current-status.md)

---

*KubeTriage is a research and engineering portfolio project by [L. A. Moss](https://github.com/la-moss), exploring what trustworthy, auditable diagnostic systems require — determinism, provenance, abstention, and evaluation methodology that can distinguish genuine evidence from evidence shaped by how the test data was selected.*
