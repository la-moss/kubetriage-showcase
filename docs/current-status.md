# KubeTriage — current status

Public governed state only. Incident contents, candidate identities, and protected evaluation material are not published here.

## Diagnostic baseline

**Frozen diagnostic tag:** `v2.8.0-a6b-pre-s4-result-inventory-reviewed-baseline`  
**Commit:** `d3a1b85e5232a36192ed55aad9f84db090e88cc4`

The diagnostic engine remains frozen at this baseline.

## Controlled evaluation

| Suite | Reviewed result |
| --- | ---: |
| Golden | 20/20 |
| Controlled holdout | 24/24 |
| Adversarial | 102/102 |
| Metamorphic | 67/67 |

Controlled holdout metrics:

| Metric | Result |
| --- | ---: |
| Top-1 accuracy | 100% |
| Brier score | 0.0557 |
| ECE | 0.2186 |
| False high confidence (≥ 0.80) | 0 |

These results describe behaviour on governed controlled corpora. Real-incident performance is being evaluated separately.

## Real-incident validation programme

Current governed state:

```text
registered candidates: 4
admitted incident families: 2
acquisition: OPEN
V2 analysis methodology: FROZEN
V2 diagnostic execution: NOT STARTED
real ground truth: 0
representability: UNADJUDICATED
evaluation cohort: ABSENT
scores/results/reveal: ABSENT
Traversal 2: NOT STARTED
```

**Acquisition protocol:** `V1-RIC-ACQ-PROTOCOL-L1-001`  
**Active version:** `1.1.0`

**Earliest Traversal-2 eligibility:** `2026-08-31T00:00:00Z`

The V2 blind real-incident analysis methodology has been independently reviewed and frozen before any V2 diagnostic result is produced.

Frozen methodology identity:

```text
bytes: 35136
SHA-256: 9a1f15dc0347d0bc7aecf0bf082e5029ea23e1426fd0405baf4e4a483ba5fb67
```

Freezing the methodology does not mean the evaluation has executed. No real incident has been run through V2, no real ground truth has been adjudicated, and no evaluation cohort, score, result, or reveal exists yet.

## Current programme

```text
frozen diagnostic baseline
        ↓
support-blind real-incident acquisition
        ↓
human admission
        ↓
ground-truth adjudication
        ↓
representability / coverage analysis
        ↓
frozen V2 methodology
        ↓
blind V2 real-incident execution
        ↓
evidence-driven revision, if justified
```

The current priority is to complete the real-incident validation path without changing the frozen diagnostic engine or evaluation method in response to eventual results.

## Technical review

The actual implementation, tests, specifications, and technical review map are available in the public main repository:

- [KubeTriage implementation](https://github.com/la-moss/KubeTriage)
- [Technical review guide](https://github.com/la-moss/KubeTriage/blob/main/docs/technical-review-guide.md)
- [Reviewed diagnostic baseline](https://github.com/la-moss/KubeTriage/tree/v2.8.0-a6b-pre-s4-result-inventory-reviewed-baseline)

See the [showcase README](../README.md) for the current project overview.
