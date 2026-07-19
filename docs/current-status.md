# KubeTriage — current status

Five-minute briefing for an engineer assessing the project after independent review through **A5B-H1**.

## What it does today

KubeTriage is a **deterministic, read-only** Kubernetes diagnostic engine. It diagnoses from **frozen replay evidence**, not live clusters. It extracts typed facts, classifies seven bounded incident classes, ranks hypotheses, scores confidence, optionally runs one drift recheck, seals an immutable run manifest, and can admit a provenance-aware explanation only after claim → fact → evidence verification.

There is **no LLM in the engine**, **no autonomous agent**, **no remediation**, and **no cluster mutation path**.

## What has passed independent review

| Gate | Status |
| --- | --- |
| A4A — run manifest and evidence identity | Passed |
| A4B — exact claim → fact → evidence provenance (through H4) | Passed |
| A5A — governed adversarial admission corpus | Passed (102/102) |
| A5B — property / metamorphic evaluation (through H1) | Passed (67/67) |
| A5 adversarial-evaluation programme | **Complete** |

## Latest evaluation numbers

| Check | Result |
| --- | ---: |
| Golden replay | 20/20 |
| Holdout replay | 24/24 |
| A5A adversarial cases | 102/102 |
| A5B metamorphic properties | 67/67 |
| Unexpected adversarial admissions | 0 |
| Capability leaks | 0 |
| Renderer leaks | 0 |
| Base top-1 | 100% |
| Brier | 0.0557 |
| ECE | 0.2186 |
| False-high confidence | 0 |
| Drift post top-1 | 100% |
| Drift flip accuracy | 100% |
| Drift non-inflation violations | 0 |

## Frozen evaluation digests

| Suite | Digest |
| --- | --- |
| A5A corpus | `60504978eaa046f9fe04b4c532ebf4c6039c74e2e42caa727312f384e370bb52` |
| A5B suite v2 | `b367b02266444c0b680b28fb97ae60c197e4e00c2673aac3285fb82fcaebd124` |

## What remains blocked

**Real-agent admission remains blocked.** Completing A5 does not admit a provider, model, or autonomous agent.

Still absent / blocked:

- real autonomous AI agent;
- provider or model integration;
- remediation;
- live production diagnosis mode;
- cluster mutation path.

Remaining gates are **operational and human**: isolation, authn/authz, credentials, audit, shadow operation, explicit approval, and kill-switch procedures.

## Honest limitations

- Seven pattern-based classes — not unrestricted Kubernetes coverage.
- Replay is the diagnostic source of truth.
- Small corpus limits statistical calibration claims; ECE is above target and not tuned away.
- Evaluation is governed adversarial and metamorphic testing — **not** formal verification.
- Digests provide deterministic integrity — **not** cryptographic authenticity.
- Envelope-local IDs such as `ev-001` in the A2 sample are **not** the current stable fact-identity model used by A4B.

## Confidence-removal distinction (A5B-H1)

- **Pure support removal** — confidence must not increase.
- **Mixed-information removal** — no universal confidence direction; the exact confidence decomposition must explain the change.

A reviewed mixed example rises from 0.72 to 0.88 after removing a tool that also carried the sole weak support fact, clearing `mixed_evidence_dampening` (0.16 → 0.0). That is not “less evidence ⇒ more confidence.”

## Next operational questions

Without announcing a new implementation phase, the practical questions are now operational rather than diagnostic-engine design:

1. What isolation and credential model would a future explanation consumer require?
2. What human approval and revocation process would gate any shadow trial?
3. How would regressions in A5 digests or replay baselines revoke temporary permission?

Until those are answered and approved, KubeTriage stays a deterministic diagnostic substrate — not an AI agent near a cluster.
