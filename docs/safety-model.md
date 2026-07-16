# KubeClaw Safety Model

KubeClaw's deterministic engine and runner safety are enforced in code, not
prompts. Future agent-output enforcement is not yet complete: real-agent
admission is currently blocked, and no real autonomous agent exists. This
document describes enforced boundaries and documented targets; it does not
claim that safety bypass by a future agent is already impossible.

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

KubeClaw targets a dedicated lab cluster context (`kind-kubeclaw`) by default. Requests against other cluster contexts are refused unless an explicit unsafe override is provided. KubeClaw will not silently target production or arbitrary clusters.

## Namespace guard

Diagnosis is scoped to permitted namespaces. Requests targeting protected namespaces (such as `kube-system`) are refused. Each session diagnoses a single namespace in a single cluster context.

## Output path guard

Evidence and session output paths are validated. Writes to governed corpus directories (golden, holdout) are not permitted from capture or generation pipelines without explicit governance review.

## Refusal as terminal state

When KubeClaw refuses a request — due to context guard, namespace guard, blocked verbs, validation failure, or other safety checks — the refusal is **terminal**:

- No diagnostic loop continues
- No diagnosis is produced
- No remediation is attempted
- No agent or LLM may override or continue past refusal

Refusal is successful safety behaviour, not an error to retry around.

## Replay as source of truth

All engine correctness is evaluated against deterministic replay of frozen evidence. Live cluster behaviour is not the evaluation authority. Generated sessions from live capture are unreviewed until human review and promotion.

## No remediation

KubeClaw produces diagnostic reports and refusals. It does not:

- Apply, patch, delete, or scale resources
- Execute commands inside containers
- Restart deployments or drain nodes
- Suggest or execute automated fixes

Write and mutation verbs are blocked unconditionally.

## No live production cluster support

KubeClaw does not target production clusters. The optional kind lab is local-only. There is no live production diagnosis mode.

## No LLM inside the engine

The diagnostic engine contains no LLM calls, no model inference, and no randomness. Classification, confidence scoring, and report generation are deterministic code.

Future LLM layers, if admitted after hardening and human approval, must sit
**outside** the engine. The target boundary permits explanation but prohibits
diagnosis, remediation, and engine-result override.

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

Any future agent admitted to consume KubeClaw output must:

- Preserve the KubeClaw diagnosis exactly (no class override, no confidence inflation)
- Respect refusals as terminal
- Avoid inventing evidence not present in the engine result
- Avoid remediation language and kubectl mutation commands
- Include a safety boundary attestation in every explanation

## Agent-safe response envelope

A strict, closed contract — `agent_safe_response/v1` — defines the only shape
of KubeClaw output a future agent would be allowed to consume:

- Exactly one mutually exclusive outcome per response: `diagnosis`,
  `insufficient_evidence`, or `refusal`.
- Required drift, evidence, safety, and provenance sections.
- No remediation, fix, action-plan, or command fields exist in the schema.
- The AI may explain but not override: the envelope is an immutable authority
  record, and agent prose would live structurally outside it.

Deterministic OpenClaw output is mapped into this envelope and validated
against the contract before being exposed; malformed envelopes fail closed.
This validation currently applies to deterministic OpenClaw-facing output
only — it is a prerequisite for real-agent admission, not real-agent admission
itself. Real-agent admission remains blocked.

The current private implementation additionally contains deterministic
string/value policy scaffolds for selected cases. They do not yet enforce
semantic evidence provenance or the complete response boundary and are not
sufficient real-agent gates. Policy hardening, stable citation/provenance
hardening, and adversarial evaluation remain future gates.
