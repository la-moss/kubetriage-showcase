# KubeTriage — current status

Public governed counters only. Incident contents, candidate identities, and protected evaluation material are not published here.

```text
registered candidates: 4
admitted incident families: 2
acquisition: OPEN
V2 blind evaluation: NOT STARTED
```

**Frozen diagnostic baseline:** `v2.8.0-a6b-pre-s4-result-inventory-reviewed-baseline`  
**Commit:** `d3a1b85e5232a36192ed55aad9f84db090e88cc4`

**Acquisition protocol:** `V1-RIC-ACQ-PROTOCOL-L1-001`  
**Active version:** `1.1.0`

Real ground-truth adjudication has not begun. The V2 analysis plan is frozen; V2 diagnostic execution has not started.

See the [README current programme](../README.md#current-programme).

---

## A6-era status briefing (historical)

> **Historical briefing.** The section below records independently reviewed A4–A6B-S2 gates and controlled-evaluation numbers. It is **not** the live corpus-counter source.
>
> **Current diagnostic authority:** frozen baseline `v2.8.0-a6b-pre-s4-result-inventory-reviewed-baseline`. Later A6 inventory work did not change diagnostic behaviour.
>
> **Current validation programme:** support-blind real-incident corpus acquisition and human admission are in progress under a frozen protocol. The V2 analysis plan is frozen; V2 diagnostic execution has not begun.
>
> Agent-admission (A6) work is paused while that validation proceeds. Completing A5 or publishing A6B-S2 did not connect a provider or admit an autonomous agent.

Five-minute historical briefing for an engineer assessing independently reviewed work through **A6B-S2** (reviewed A6 lifecycle baseline tag `v2.2.0-a6b-s2-reviewed-baseline`).

## What it does today

KubeTriage is a **deterministic, read-only diagnostic engine**. It diagnoses from **frozen replay evidence**, not live clusters. Today's implemented evidence path is primarily **Kubernetes-native** (six-tool allowlist replay). It extracts typed facts, classifies seven bounded incident classes, ranks hypotheses, scores confidence, optionally runs one drift recheck, seals an immutable run manifest, and can admit a provenance-aware explanation only after claim → fact → evidence verification.

KubeTriage sits in the diagnostic layer: it validates evidence, extracts facts, ranks hypotheses, calculates confidence, and produces replayable results. It is **not** an observability, monitoring, or telemetry platform, and it does not replace systems that produce evidence.

The **A6 controlled explanation-agent admission** programme produced the reviewed constitution and lifecycle baselines below. The latest reviewed A6 milestone in this briefing is **A6B-S2** (request lifecycle ledger, authority binding, and idempotency). That work governs how a future explanation consumer would prepare, register, and replay admission requests — it does **not** connect a provider or admit a real agent. A6 is paused; it is not the current validation programme.

There is **no LLM in the engine**, **no admitted autonomous agent in KubeTriage**, **no remediation**, and **no cluster mutation path**.

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

## A6 programme (historical / paused)

A6 is the controlled programme for deciding whether explanation-only admission can ever be opened. It does not add remediation, live diagnosis, provider SDKs, or cluster writes. Current project focus is V1 real-incident validation, not A6 completion.

| Phase | Scope | Status |
| --- | --- | --- |
| A6A | Versioned admission constitution (permissions, prohibitions, admitted claims) | **Complete** |
| A6B | Request lifecycle and isolation (preparation, ledger, idempotency, bounded transitions) | **Paused** — S2 published; S3+ blocked pending review |
| A6C | Offline hostile-provider harness (mock/replay only) | Not started |
| A6D | Authentication, authorisation and tenancy | Not started |
| A6E | Audit, revocation and operational controls | Not started |
| A6F | Real-provider shadow mode | Not started |
| A6G | Human-approved explanation pilot | Not started |

**Latest reviewed A6 lifecycle baseline in this briefing:** `v2.2.0-a6b-s2-reviewed-baseline` (A6B-S2 ledger and authority binding). **Current diagnostic baseline:** `v2.8.0-a6b-pre-s4-result-inventory-reviewed-baseline`.

Frozen A6 identities (unchanged across S1/S2):

| Identity | Digest |
| --- | --- |
| `kubetriage_agent_admission_constitution/v1` | `0ce4008cbf65a04a49466399f3eb891fcd20d5855b7885be589610203d541745` |
| `kubetriage_lifecycle_policy/v1` | `694b6d9424c5e0d48831ba374399b23025982266dbaedc311b8cc6ed12a282ae` |

## What remains blocked

**Real-agent admission remains blocked.** Completing A5 or publishing A6B-S2 does not admit a provider, model, or autonomous agent.

Still absent / blocked:

- autonomous agent connected to or admitted by KubeTriage;
- provider or model integration;
- remediation;
- live production diagnosis mode;
- cluster mutation path;
- non-Kubernetes governed evidence adapters (architectural scope only);
- A6B-S3 capacity/serial execution (draft architecture only);
- A6C–A6G operational gates.

Remaining gates are **operational and human**: isolation, authn/authz, credentials, audit, shadow operation, explicit approval, and kill-switch procedures.

## Honest limitations

- Seven pattern-based classes are the current proof point — not unrestricted Kubernetes coverage and not the permanent architectural boundary.
- Replay is the diagnostic source of truth; only Kubernetes-native governed evidence is implemented today.
- Broader multi-source diagnosis remains architectural scope, not current capability.
- Small corpus limits statistical calibration claims; ECE is above target and not tuned away.
- Real-incident corpus acquisition is in progress; the corpus is not yet complete. The V2 analysis plan is frozen; V2 diagnostic execution has not begun. Real-world diagnostic reliability has not yet been established.
- Evaluation on golden / holdout / adversarial / metamorphic material is governed testing — **not** formal verification and **not** real-incident validation.
- Digests provide deterministic integrity — **not** cryptographic authenticity.
- Envelope-local IDs such as `ev-001` in the A2 sample are **not** the current stable fact-identity model used by A4B.

## Confidence-removal distinction (A5B-H1)

- **Pure support removal** — confidence must not increase.
- **Mixed-information removal** — no universal confidence direction; the exact confidence decomposition must explain the change.

A reviewed mixed example rises from 0.72 to 0.88 after removing a tool that also carried the sole weak support fact, clearing `mixed_evidence_dampening` (0.16 → 0.0). That is not “less evidence ⇒ more confidence.”

## Next operational questions (historical A6 framing)

The diagnostic engine and A5 evaluation programme are complete. A6B-S2 closed ledger and idempotency for the lifecycle runtime. Those A6 questions remain blocked and paused:

1. When A6B-S3+ (capacity, serial execution, rendering custody) pass review, does the lifecycle runtime remain process-local and fail-closed?
2. What isolation and credential model would a future explanation consumer require (A6D)?
3. What human approval and revocation process would gate any shadow trial (A6F/A6G)?
4. How would regressions in A5 digests, A6 constitution digests, or replay baselines revoke temporary permission?

**Current programme:** support-blind real-incident acquisition and human admission under the frozen protocol, with the diagnostic engine held at the v2.8 baseline. Until A6G and explicit human approval, KubeTriage stays a deterministic diagnostic substrate — not an AI agent near a cluster.
