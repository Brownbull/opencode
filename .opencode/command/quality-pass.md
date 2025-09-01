---
name: /quality-pass
description: Plan and validate quality, usability, accessibility, and performance gates for a set of implemented or planned slices.
agent: qra
mode: analysis
version: 0.1.0
inputs:
  required:
    - scope
  optional:
    - slice_ids
    - target_metrics
    - recent_defects
    - adoption_signals
constraints:
  - Enforce validation_batch_units (agent_limits)
  - Respect max_new_concepts_per_iteration (no uncontrolled metric proliferation)
  - Derive scenarios from LUA reality signals (no synthetic only)
  - Accessibility baseline ≥ WCAG-A (raise escalation if unmet)
outputs:
  - quality_matrix
  - test_plan
  - accessibility_report
  - defect_risk_profile
escalation_triggers:
  - ACCESSIBILITY_CRITICAL
  - PERFORMANCE_REGRESSION
  - ADOPTION_GAP
  - PACING_VIOLATION
success_criteria:
  - Each slice/test case maps to explicit acceptance_criteria_ref
  - Time-to-complete targets present for primary user actions
  - Localization / Spanish coverage explicitly stated
---

# /quality-pass Command Specification

## Purpose
Generate a deterministic quality validation plan for a defined `scope` (single slice, set of slices, or release bundle) ensuring usability, accessibility, performance, and localization gates are measurable before deployment.

## Required Preconditions
1. Slice specs or implementation artifacts available (from MAKA) OR planning slices provided.
2. VIVA acceptance criteria accessible for referenced items.
3. LUA adoption / friction signals (at least latest summary) if scope != single slice.
4. `human_context` + `agent_limits` for pacing and batch sizing (validation_batch_units).

## Scope Interpretation
- scope = `slice:ID` → single slice focus
- scope = `bundle:NAME` → group of slice_ids required
- scope = `release` → aggregate across current ready-for-deploy set

## Procedure
1. Validate presence of required artifacts (slice specs, acceptance criteria, agent_limits).
2. Segment test planning into batches ≤ validation_batch_units.
3. For each slice (or aggregated feature path):
   - Extract primary user action(s) & target time (seconds).
   - Map acceptance_criteria_ref → test cases.
   - Determine cognitive load factors (click count, field count).
   - Assign accessibility baseline; flag gaps.
4. Build quality_matrix; link each test to metric dimension: (usability|performance|accessibility|localization|regression).
5. Identify historical defects (recent_defects) influencing risk weighting.
6. Generate test_plan with ordering by (risk DESC, user_frequency DESC, complexity ASC).
7. Compile accessibility_report (issues sorted by severity).
8. Produce defect_risk_profile with probability × impact scoring (deterministic weight table).

## Quality Matrix Schema
| id | slice_id | test_title | dimension | acceptance_criteria_ref | target_metric | threshold | risk_weight | batch_id |

## Test Plan Schema (YAML)
```
test_plan:
  batches:
    - batch_id: B1
      tests: [Q1, Q2, Q3]
      rationale: "High risk + high frequency"
    - batch_id: B2
      tests: [...]
plan_constraints:
  validation_batch_units: <n>
  total_batches: <n>
```

## Accessibility Report Schema
| issue_id | element | severity | rule | description | remediation | slice_id |

## Defect Risk Profile Schema
| slice_id | area | probability | impact | risk_score | mitigation_reference |

Probability & impact deterministic mapping (e.g., probability from historical defect density or complexity class).

## Failure Block (Standard)
```
## Artifact:
(in-memory)
Status: FAILED
Missing: [scope|slice_ids|acceptance_criteria|agent_limits]
Deferred: [ ... ]
Escalations: [ACCESSIBILITY_CRITICAL|PACING_VIOLATION|...]
```

## Success Block (Standard)
```
## Artifact:
.opencode/output/quality-pass/<scope_hash>.md
Status: SUCCESS
Missing: []
Deferred: [ ... ]
Escalations: [ ... ]
```

## Determinism Rules
- risk_score = probability_weight + impact_weight (integer weights table versioned)
- Sorting: risk_score DESC → dimension lexicographic → id
- Batch assignment stable via round-robin fill respecting validation_batch_units.

## Escalation Conditions
| Escalation | Condition |
|------------|-----------|
| ACCESSIBILITY_CRITICAL | Blocking WCAG-A failure on primary task path |
| PERFORMANCE_REGRESSION | Latency/interaction > target_metric threshold |
| ADOPTION_GAP | LUA signals show usage below adoption threshold for target action |
| PACING_VIOLATION | New metric categories > max_new_concepts_per_iteration |

## Example Invocation
```
/quality-pass scope=slice:BK-012 slice_ids=BK-012
```

## Example Quality Matrix (Abbrev)
```
| id  | slice_id | test_title            | dimension     | acceptance_criteria_ref | target_metric | threshold | risk_weight | batch_id |
| Q1  | BK-012   | Create patient <30s   | usability     | AC-BK-012-1             | time_seconds  | 30        | 8           | B1       |
| Q2  | BK-012   | Form field labels es  | localization  | AC-BK-012-1             | coverage_pct  | 100       | 5           | B1       |
| Q3  | BK-012   | Color contrast check  | accessibility | AC-BK-012-1             | passes        | 100       | 7           | B1       |
```

## Non-Goals
- Implementing fixes
- Architecture tuning (delegate to SYRA)
- Generating synthetic UX scenarios not traceable to LUA signals

End of /quality-pass specification.
