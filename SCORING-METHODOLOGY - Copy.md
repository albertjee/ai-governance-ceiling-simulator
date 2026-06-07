# Scoring Methods

Document Revision: REV 1.1  
Simulator Version: v1.5  
Last Updated: 2026-06-07

## Purpose and Limitations

The AI Security Governance Ceiling Simulator is an executive what-if model. It estimates how governance maturity can cap AI security capability and allows users to test policy, ownership, evidence, lifecycle, logging, and response improvements.

The simulator describes itself as a public heuristic simulator. Its scores are heuristic and are rounded to two decimals for display. It does not perform an evidence-based tenant assessment.

## Scoring Inputs

### Starting Maturity

| Internal ID | Label | Default | Range | Step |
|---|---|---:|---:|---:|
| `startingMaturity` | Starting maturity estimate | 1.75 | 1.00 to 5.00 | 0.05 |

The HTML input ID and diagnostic input ID are `startingMaturity`. Its value is stored internally as `state.starting`. The raw-capability formula uses `state.starting`; these names refer to the same current starting-maturity input at different implementation layers.

### Domain Maturity

| Internal ID | Label | Default | Weight |
|---|---|---:|---:|
| `governance` | Governance | 1.60 | 0.28 |
| `technical` | Technical Capability | 1.90 | 0.22 |
| `evidence` | Evidence Confidence | 1.45 | 0.20 |
| `nhi` | NHI Lifecycle | 1.35 | 0.16 |
| `logging` | Logging / Traceability | 1.50 | 0.14 |

Each domain slider has a range of 1.00 to 5.00 and a step of 0.05.

### Improvement Levers

| Internal ID | Label | Target Domain | Contribution | Default Selected |
|---|---|---|---:|---|
| `aup` | Ratified AI Acceptable Use Policy | `governance` | 0.28 | No (`false`) |
| `riskOwner` | Named AI risk owner | `governance` | 0.42 | No (`false`) |
| `council` | AI Governance Council | `governance` | 0.36 | No (`false`) |
| `riskRegister` | AI risk register | `evidence` | 0.36 | No (`false`) |
| `agentInventory` | AI agent inventory | `nhi` | 0.34 | No (`false`) |
| `nhiOwnerMapping` | NHI owner mapping | `nhi` | 0.36 | No (`false`) |
| `siem` | SIEM/logging evidence | `logging` | 0.44 | No (`false`) |
| `playbook` | AI incident-response playbook | `technical` | 0.32 | No (`false`) |

When selected, a lever adds its contribution to its target domain. The adjusted domain is clamped to the range 1.00 to 5.00:

```text
values[item.domain] = clamp(values[item.domain] + item.impact, 1, 5)
```

The diagnostic's known potential contribution is the configured lever contribution. Its applied contribution is the effective adjusted-domain increase after this existing clamp.

An individual lever's standalone final-score contribution is unavailable. Its effect is embedded in the adjusted domain score and may then affect the weighted score, penalties, caps, and final score.

### Lever Application Order

`adjustedDomains()` iterates the `improvements` array with `improvements.forEach(...)`. Selected levers are therefore applied in this exact order:

1. `aup` - Ratified AI Acceptable Use Policy
2. `riskOwner` - Named AI risk owner
3. `council` - AI Governance Council
4. `riskRegister` - AI risk register
5. `agentInventory` - AI agent inventory
6. `nhiOwnerMapping` - NHI owner mapping
7. `siem` - SIEM/logging evidence
8. `playbook` - AI incident-response playbook

Each selected lever is clamped immediately after it is added. Application order does not change the final adjusted domain total for the current positive contributions, but it can change each lever's diagnostic `appliedDomainContribution` when multiple levers target a domain that reaches the 5.00 clamp. Earlier levers can receive their full applied contribution while a later lever receives only the remaining amount before the clamp, or zero.

## Canonical Calculation Order

The implemented scoring and result pipeline is:

1. Read `state.starting`, `state.domains`, and `state.improvements`.
2. Copy current domain inputs in `adjustedDomains()`.
3. Apply selected improvement levers in the exact `improvements` array order.
4. Clamp the target domain immediately after each selected lever contribution.
5. Calculate the weighted domain score through `scoreFor(values)`.
6. Calculate `rawCapability`.
7. Calculate `evidencePenalty` and `tracePenalty`.
8. Calculate and clamp `evidenceAdjusted`.
9. Calculate `governanceCap`, `nhiCap`, and `traceCap`.
10. Calculate the final `capped` score.
11. Build and sort blockers by adjusted domain value.
12. Determine `activeCeiling`.
13. Calculate and rank possible moves by temporarily enabling each currently disabled lever.
14. In `updateResults()`, determine the maturity stage and render result values.
15. Calculate graph state from adjusted domains and the final capped score.
16. Publish the scoring diagnostic.

## Scoring Formulas

The formulas below reproduce the expressions implemented in `index.html`.

### Clamp

```text
Math.min(max, Math.max(min, value))
```

### Weighted Domain Score

```text
domains.reduce((sum, domain) => sum + values[domain.id] * domain.weight, 0)
```

Each weighted contribution is:

```text
adjusted domain value * domain weight
```

### Raw Capability

```text
(scoreFor(values) * 0.72) + (state.starting * 0.28)
```

### Penalties

Evidence penalty:

```text
Math.max(0, 2.75 - values.evidence) * 0.18
```

Traceability penalty:

```text
Math.max(0, 2.6 - values.logging) * 0.1
```

### Evidence-Adjusted Score

```text
clamp(rawCapability - evidencePenalty - tracePenalty, 1, 5)
```

### Caps

Governance cap:

```text
clamp(values.governance + 1.0, 1.55, 5)
```

NHI lifecycle cap:

```text
clamp(values.nhi + 1.04, 1.75, 5)
```

Logging / traceability cap:

```text
clamp(values.logging + 1.12, 1.75, 5)
```

### Final Capped Score

```text
Math.min(evidenceAdjusted, governanceCap, nhiCap, traceCap)
```

### Active Ceiling

```text
capped + 0.01 < evidenceAdjusted
```

The displayed ceiling delta is:

```text
evidenceAdjusted - capped
```

## Rounding Rules

- Scoring calculations retain JavaScript numeric precision. No intermediate scoring value is rounded before use in a later scoring formula.
- `format(value)` uses `Number(value).toFixed(2)` for UI and diagnostic display values.
- The diagnostic preserves full-precision values as `rawValue` and formatted two-decimal values as `displayValue`.
- `tiedBlockers` uses equality of the two-decimal `displayValue` strings.
- The final capped score retains full internal precision before it is formatted for display.

## Maturity Stage

| Final Score Condition | Stage |
|---|---|
| `score >= 4.5` | Stage 5 |
| `score >= 3.5` | Stage 4 |
| `score >= 2.5` | Stage 3 |
| `score >= 1.75` | Stage 2 |
| Otherwise | Stage 1 |

## Blocker Selection

The blocker candidates use adjusted domain values and are sorted in ascending value order:

```text
blockers.sort((a, b) => a.value - b.value)
```

The first sorted blocker is the primary blocker:

```text
blockers.sort((a, b) => a.value - b.value)[0]
```

| Internal ID | Label | Unlock | Detail |
|---|---|---|---|
| `governance` | Governance | Assign named AI risk ownership | Ownership and council cadence raise the governance cap. |
| `evidence` | Evidence Confidence | Create an AI risk register | Evidence confidence reduces the discount applied to capability claims. |
| `nhi` | NHI Lifecycle | Map NHI owners and maintain AI agent inventory | NHI lifecycle maturity limits defensibility for autonomous and agentic AI. |
| `logging` | Logging / Traceability | Add SIEM/logging evidence | Traceability makes AI control operation visible and reviewable. |
| `technical` | Technical Capability | Test the AI incident-response playbook | Response practice raises operational readiness. |

The `tiedBlockers` diagnostic does not change primary-blocker selection. It includes every ordered blocker whose two-decimal display value equals the primary blocker's two-decimal display value:

```text
blocker.value.toFixed(2) === primaryBlocker.value.toFixed(2)
```

## Possible-Move Calculations

For each disabled improvement lever, the simulator temporarily enables that lever and recalculates adjusted domains, raw capability, evidence penalty, traceability penalty, evidence-adjusted score, and final capped score using the same formulas above.

The estimated score lift is:

```text
Math.max(0, nextCapped - capped)
```

Possible moves are sorted by descending gain and limited to the first three:

```text
.sort((a, b) => b.gain - a.gain)
.slice(0, 3)
```

Only these ranked top-three possible-move gains are available as standalone diagnostic values. Standalone final-score contributions for all individual levers are otherwise unavailable because lever effects are embedded in the scoring pipeline.

## Graph-State Calculations

The domain maturity profile uses adjusted domain values.

Graph position:

```text
clamp(((value - 1) / 4) * 100, 0, 100)
```

The graph scale is 1.00 to 5.00. Its ticks are 1.75, 2.50, 3.50, and 4.50. The final-score marker uses the final capped score.

The weakest graph domain is the first domain retained while reducing adjusted values with a strict lower-than comparison. The strongest graph domain is the first domain retained while reducing adjusted values with a strict greater-than comparison.

## Worked Baseline Example

The baseline uses the defaults in `index.html`: `state.starting` is 1.75, domain inputs use their configured defaults, and every improvement lever is `false`.

### Inputs and Adjusted Domains

Because no levers are selected, adjusted domains equal input domains:

| Domain | Input | Adjusted |
|---|---:|---:|
| Governance | 1.60 | 1.60 |
| Technical Capability | 1.90 | 1.90 |
| Evidence Confidence | 1.45 | 1.45 |
| NHI Lifecycle | 1.35 | 1.35 |
| Logging / Traceability | 1.50 | 1.50 |

### Weighted Contributions

| Domain | Calculation | Contribution |
|---|---|---:|
| Governance | `1.60 * 0.28` | 0.448 |
| Technical Capability | `1.90 * 0.22` | 0.418 |
| Evidence Confidence | `1.45 * 0.20` | 0.290 |
| NHI Lifecycle | `1.35 * 0.16` | 0.216 |
| Logging / Traceability | `1.50 * 0.14` | 0.210 |

Weighted domain score:

```text
0.448 + 0.418 + 0.290 + 0.216 + 0.210 = 1.582
```

Starting-maturity contribution:

```text
state.starting * 0.28 = 1.75 * 0.28 = 0.49
```

Raw capability:

```text
(1.582 * 0.72) + (1.75 * 0.28)
= 1.13904 + 0.49
= 1.62904
```

Evidence penalty:

```text
Math.max(0, 2.75 - 1.45) * 0.18
= 1.30 * 0.18
= 0.234
```

Traceability penalty:

```text
Math.max(0, 2.6 - 1.50) * 0.1
= 1.10 * 0.1
= 0.110
```

Evidence-adjusted score:

```text
clamp(1.62904 - 0.234 - 0.110, 1, 5)
= clamp(1.28504, 1, 5)
= 1.28504
```

Caps:

```text
Governance cap = clamp(1.60 + 1.0, 1.55, 5) = 2.60
NHI lifecycle cap = clamp(1.35 + 1.04, 1.75, 5) = 2.39
Logging / traceability cap = clamp(1.50 + 1.12, 1.75, 5) = 2.62
```

Final score:

```text
Math.min(1.28504, 2.60, 2.39, 2.62) = 1.28504
```

| Baseline Result | Value |
|---|---|
| Full-precision final score | 1.28504 |
| Displayed final score | 1.29 |
| Maturity stage | Stage 1 |
| Active ceiling | `false`, because `1.28504 + 0.01 < 1.28504` is false |
| Primary blocker | NHI Lifecycle at 1.35 |
| `tiedBlockers` | NHI Lifecycle only |

The primary blocker is selected from the lowest adjusted domain value, independently of which value determines the final capped score. At baseline, the final score equals the evidence-adjusted score, while NHI Lifecycle remains the lowest domain and therefore the primary blocker.

## Diagnostic Trace and JSON Export

Every call to `updateResults()` calculates the current score and publishes one complete immutable diagnostic object. This occurs on:

- Initial page load
- Starting-maturity slider input
- Every domain slider input
- Every improvement toggle change
- Reset to baseline

The same current diagnostic object is:

- Printed automatically to the browser console with the label `AI Governance Simulator Scoring Diagnostic`
- Displayed in the read-only Scoring Trace
- Exposed through `window.AI_GOVERNANCE_SIMULATOR_SCORE_TRACE`
- Returned by `window.getScoringDiagnostic()`
- Downloaded by `window.downloadScoringDiagnostic()`
- Downloaded by the Export JSON button

The three `window` diagnostic interfaces are defined as read-only properties. The current diagnostic object and its nested values are frozen.

The diagnostic includes:

- `schemaVersion: "1.0"`
- `simulatorVersion: "v1.5"`
- `exportType: "Scoring System Diagnostic Export"`
- UTC `generatedUtc`
- Internal IDs and human-readable labels
- Full-precision `rawValue` and two-decimal `displayValue` for diagnostic scoring values
- Current inputs and domains
- Selected and disabled levers
- Known potential and applied lever contributions
- Weighted contributions and intermediate calculations
- Penalties, caps, final score, and limiting caps
- Primary blocker and `tiedBlockers`
- Graph-state values
- Ranked possible moves
- Required disclaimers

Downloads use this UTC filename pattern:

```text
ai-governance-scoring-trace-v1.5-YYYYMMDD-HHMMSS.json
```

## Exact Disclaimers

Non-assessment disclaimer:

> This is a heuristic what-if simulator, not an evidence-based tenant assessment.

Non-affiliation disclaimer:

> Not affiliated with, endorsed by, or representative of SANS, Microsoft, or Microsoft Secure Score.
