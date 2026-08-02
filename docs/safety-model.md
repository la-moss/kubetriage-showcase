# KubeTriage Safety Model

KubeTriage's deterministic engine and runner safety are enforced in code, not
prompts. A4A/A4B provenance binding and A5A/A5B adversarial evaluation have
passed independent review, and the A5 programme is complete. **A6A** (admission
constitution) and **A6B-S2** (lifecycle ledger and idempotency) have passed
independent review and publication. **Real-agent admission remains blocked**.
No real autonomous agent, provider integration, remediation path, or cluster
mutation capability exists.

This document describes enforced boundaries and remaining operational/human
gates. It does not claim that every future agent-output bypass is already
impossible in a production deployment.

## Read-only evidence capture

All diagnostic inputs come from frozen tool outputs. The engine never executes write or mutation operations against a cluster during diagnosis. Evidence capture (when used) is restricted to read-only kubectl commands mapped to the six logical tools.

## Six logical tools

Exactly six read-only diagnostic tools are permitted:

| Tool | Purpose |
| --- | --- |
| `events_tail` | Recent namespace events |
| `describe_pod` | Pod status and conditions |
| `describe_deploy` | Deployment status |
| `logs` | Container log output |
| `get_yaml` | Resource YAML |
| `top_pod` | Resource usage metrics |

The allowlist is asserted in code. Unknown tools, spoofed tool names, and homoglyph attacks are rejected.

## Context guard

KubeTriage targets a dedicated lab cluster context (`kind-kubetriage`) by default. Requests against other cluster contexts are refused unless an explicit unsafe override is provided. KubeTriage will not silently target production or arbitrary clusters.

## Namespace guard

Diagnosis is scoped to permitted namespaces. Requests targeting protected namespaces (such as `kube-system`) are refused. Each session diagnoses a single namespace in a single cluster context.

## Output path guard

Evidence and session output paths are validated. Writes to governed corpus directories (golden, holdout) are not permitted from capture or generation pipelines without explicit governance review.

## Refusal as terminal state

When KubeTriage refuses a request — due to context guard, namespace guard, blocked verbs, validation failure, or other safety checks — the refusal is **terminal**:

- No diagnostic loop continues
- No diagnosis is produced
- No remediation is attempted
- No agent or LLM may override or continue past refusal

Refusal is successful safety behaviour, not an error to retry around.

## Replay as source of truth

All engine correctness is evaluated against deterministic replay of frozen evidence. Live cluster behaviour is not the evaluation authority. Generated sessions from live capture are unreviewed until human review and promotion.

## No remediation

KubeTriage produces diagnostic reports and refusals. It does not:

- Apply, patch, delete, or scale resources
- Execute commands inside containers
- Restart deployments or drain nodes
- Suggest or execute automated fixes

Write and mutation verbs are blocked unconditionally.

## No live production cluster support

KubeTriage does not target production clusters. The optional kind lab is local-only. There is no live production diagnosis mode.

## No LLM inside the engine

The diagnostic engine contains no LLM calls, no model inference, and no randomness. Classification, confidence scoring, and report generation are deterministic code.

Future LLM layers, if ever admitted after operational isolation review and human
approval, must sit **outside** the engine. The target boundary permits
explanation but prohibits diagnosis, remediation, and engine-result override.
The offline LLM adapter scaffold is **not** an admitted path.

## Provenance-aware admission

The current provenance-aware explanation path verifies claim → fact → evidence
binding, contribution replay, and support-set identity before minting an
immutable capability for the deterministic renderer. Failures fail closed; there
is no silent fallback to a weaker policy.

Governed adversarial (A5A) and metamorphic (A5B) evaluation exercises that
boundary. Passing those suites does not equal real-agent admission.

## A6 admission constitution (A6A milestone)

**A6A** freezes a versioned admission constitution (`kubetriage_agent_admission_constitution/v1`) that defines:

- permitted and prohibited claim types for a future explanation layer;
- refusal and insufficient-evidence invariants that must be preserved;
- that A6A is the sole semantic admission authority — lifecycle code cannot override diagnosis, confidence, facts, or provenance.

The constitution does not grant tool use, remediation, or provider access. Real-agent admission remains blocked.

## A6 lifecycle runtime (A6B through S2)

**A6B** adds process-local lifecycle controls for a future explanation consumer:

| Slice | Scope | Status |
| --- | --- | --- |
| S1 | Canonical admission-request preparation and verification | Published (`v2.1.0-a6b-s1-reviewed-baseline`) |
| S2 | Ledger, authority binding, idempotent replay, bounded transitions | Published (`v2.2.0-a6b-s2-reviewed-baseline`) |
| S3+ | Capacity slot, serial execution, rendering custody | Blocked (draft architecture only) |

Lifecycle controls are fail-closed: malformed requests, conflicts, and ledger corruption produce terminal refusals — not degraded behaviour. The lifecycle runtime does not connect to a provider, expose cluster credentials, or perform remediation.

Frozen lifecycle-policy digest: `694b6d9424c5e0d48831ba374399b23025982266dbaedc311b8cc6ed12a282ae`.

## Generated sessions are unreviewed until promoted

Live-capture pipelines write to generated session directories. These sessions:

- Are not automatically part of any evaluation corpus
- Must be reviewed and redacted before promotion
- Require explicit governance approval to enter golden or holdout

## Golden and holdout are governed corpora

| Corpus | Governance |
| --- | --- |
| Golden | Development regression; may be adjusted during development with documented reason |
| Holdout | Evaluation-only; engine behaviour must not be tuned against holdout results |
| Generated | Unreviewed; not evaluation material until promoted |

Non-drift and drift metrics are reported separately and never merged as a single headline metric.

## Agent and LLM safety boundaries

Any future agent admitted to consume KubeTriage output must:

- Preserve the KubeTriage diagnosis exactly (no class override, no confidence inflation)
- Respect refusals as terminal
- Avoid inventing evidence not present in the engine result
- Avoid remediation language and kubectl mutation commands
- Include a safety boundary attestation in every explanation

## Agent-safe response envelope (A2 milestone)

A strict, closed contract — `agent_safe_response/v1` — defines an earlier
authority-envelope shape:

- Exactly one mutually exclusive outcome per response: `diagnosis`,
  `insufficient_evidence`, or `refusal`.
- Required drift, evidence, safety, and provenance sections.
- No remediation, fix, action-plan, or command fields exist in the schema.
- The AI may explain but not override: the envelope is an immutable authority
  record, and agent prose would live structurally outside it.

Deterministic OpenClaw output can be mapped into this envelope and validated
before exposure. Envelope-local evidence IDs such as `ev-001` are **not** the
current A4B stable fact-identity model.

Later **A4B v3** provenance-aware admission (`agent_explanation/v3` under
`agent_output_policy/v3`) supersedes this envelope for the admitted
provenance-bound explanation path. **A6A/A6B** add constitution and lifecycle
controls on top of that path. Real-agent admission remains blocked by
operational and human gates even after A5 programme completion and A6B-S2
publication.
