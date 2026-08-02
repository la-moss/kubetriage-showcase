# KubeTriage — current status

Five-minute briefing for an engineer assessing the project after independent review through **A6B-S2** (reviewed baseline tag `v2.2.0-a6b-s2-reviewed-baseline`).

## What it does today

KubeTriage is a **deterministic, read-only** Kubernetes diagnostic engine. It diagnoses from **frozen replay evidence**, not live clusters. It extracts typed facts, classifies seven bounded incident classes, ranks hypotheses, scores confidence, optionally runs one drift recheck, seals an immutable run manifest, and can admit a provenance-aware explanation only after claim → fact → evidence verification.

The **A6 controlled explanation-agent admission** programme is now in progress. The latest reviewed milestone is **A6B-S2** (request lifecycle ledger, authority binding, and idempotency). That work governs how a future explanation consumer would prepare, register, and replay admission requests — it does **not** connect a provider or admit a real agent.

There is **no LLM in the engine**, **no autonomous agent**, **no remediation**, and **no cluster mutation path**.

## What has passed independent review

| Gate | Status |
| --- | --- |
| A4A — run manifest and evidence identity | Passed |
| A4B — exact claim → fact → evidence provenance (through H4) | Passed |
| A5A — governed adversarial admission corpus | Passed (102/102) |
| A5B — property / metamorphic evaluation (through H1) | Passed (67/67) |
| A5 adversarial-evaluation programme | **Complete** |
| A6A — agent admission constitution | **Complete** (A6A-H1 passed) |
| A6B-S1 — isolated admission request preparation | **Published** (`v2.1.0-a6b-s1-reviewed-baseline`) |
| A6B-S2 — ledger, authority binding and idempotency | **Published** (`v2.2.0-a6b-s2-reviewed-baseline`) |

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

## A6 programme (in progress)

A6 is the controlled programme for deciding whether explanation-only admission can ever be opened. It does not add remediation, live diagnosis, provider SDKs, or cluster writes.

| Phase | Scope | Status |
| --- | --- | --- |
| A6A | Versioned admission constitution (permissions, prohibitions, admitted claims) | **Complete** |
| A6B | Request lifecycle and isolation (preparation, ledger, idempotency, bounded transitions) | **In progress** — S2 published; S3+ blocked pending review |
| A6C | Offline hostile-provider harness (mock/replay only) | Not started |
| A6D | Authentication, authorisation and tenancy | Not started |
| A6E | Audit, revocation and operational controls | Not started |
| A6F | Real-provider shadow mode | Not started |
| A6G | Human-approved explanation pilot | Not started |

**Latest reviewed baseline:** `v2.2.0-a6b-s2-reviewed-baseline` (A6B-S2 ledger and authority binding).

Frozen A6 identities (unchanged across S1/S2):

| Identity | Digest |
| --- | --- |
| `kubetriage_agent_admission_constitution/v1` | `0ce4008cbf65a04a49466399f3eb891fcd20d5855b7885be589610203d541745` |
| `kubetriage_lifecycle_policy/v1` | `694b6d9424c5e0d48831ba374399b23025982266dbaedc311b8cc6ed12a282ae` |

## What remains blocked

**Real-agent admission remains blocked.** Completing A5 or publishing A6B-S2 does not admit a provider, model, or autonomous agent.

Still absent / blocked:

- real autonomous AI agent;
- provider or model integration;
- remediation;
- live production diagnosis mode;
- cluster mutation path;
- A6B-S3 capacity/serial execution (draft architecture only);
- A6C–A6G operational gates.

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

The diagnostic engine and A5 evaluation programme are complete. A6B-S2 closes ledger and idempotency for the lifecycle runtime; the practical questions ahead are lifecycle completion and operational admission:

1. When A6B-S3+ (capacity, serial execution, rendering custody) pass review, does the lifecycle runtime remain process-local and fail-closed?
2. What isolation and credential model would a future explanation consumer require (A6D)?
3. What human approval and revocation process would gate any shadow trial (A6F/A6G)?
4. How would regressions in A5 digests, A6 constitution digests, or replay baselines revoke temporary permission?

Until A6G and explicit human approval, KubeTriage stays a deterministic diagnostic substrate — not an AI agent near a cluster.
