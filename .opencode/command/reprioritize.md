---
name: /reprioritize
description: Recompute deterministic ranked backlog using new friction/adoption signals and quality / architecture escalations.
agent: viva
mode: analysis
version: 0.1.0
inputs:
  required:
    - reason
  optional:
    - new_signals
    - previous_value_map
    - backlog_overrides
    - escalation_flags
constraints:
  - Deterministic scoring (stable weights)
  - Enforce max_new_concepts_per_iteration (category introductions)
  - Decompose epics exceeding preferred_chunk_size
  - Reject items lacking traceable pain/metric source
outputs:
  - ranked_backlog
  - scoring_matrix
  - churn_report
  - deferred_items
escalation_triggers:
  - EXCESSIVE_CHURN
  - MISSING_SIGNALS
  - PACING_VIOLATION
success_criteria:
  - Each backlog row has value_score components + source refs
  - Churn (top10 reorder %) computed vs previous map
  - Deferred list enumerated with pacing or evidence gaps
---

# /reprioritize Command Specification

## Purpose
Produce an updated deterministic ranked backlog when triggered by new LUA friction/adoption signals, QRA quality outcomes, SYRA escalations, or time‑boxed review cadence.

## Required Preconditions
1. At least one new signal OR explicit `reason=scheduled`.
2. Access to prior ranked backlog (previous_value_map) if exists for churn analysis.
3. `human_context` + `agent_limits` (pacing thresholds & preferred_chunk_size).
4. All candidate backlog items have source evidence (pain signal id, metric delta, or user quote hash).

## Procedure
1. Collect input signals: new_signals (LUA/QRA/SYRA), escalation_flags.
2. Normalize items:
   - Remove obsolete (no longer backed by active signal & not in-progress).
   - Decompose oversized items > preferred_chunk_size.
3. For each item compute components (deterministic):
   - Impact
   - AdoptionMomentum
   - FrictionSeverity
   - StrategicAlignment
   - Complexity
   - Risk
4. value_score = (Impact*W_i)+(AdoptionMomentum*W_a)+(FrictionSeverity*W_f)+(StrategicAlignment*W_s)-(Complexity*W_c)-(Risk*W_r)
5. Enforce pacing:
   - Count new categories; if > max_new_concepts_per_iteration defer overflow.
6. Sort items by (value_score DESC, Risk ASC, Complexity ASC, id ASC).
7. Compute churn_report (top10 difference vs previous).
8. Emit ranked_backlog artifact + deferred_items + any escalations.

## Ranked Backlog Schema
| rank | id | title | value_score | impact | adoption | friction | strategic | complexity | risk | sources | acceptance_criteria_ref |

## Scoring Matrix (YAML)
```
scoring_matrix:
  weights:
    impact: W_i
    adoption: W_a
    friction: W_f
    strategic: W_s
    complexity: W_c
    risk: W_r
  version: 1
```

## Churn Report Schema
| metric | value |
|--------|-------|
| top10_reorder_pct | n% |
| added_items | k |
| removed_items | m |
| deferred_items | d |

If top10_reorder_pct > threshold → EXCESSIVE_CHURN escalation.

## Deferred Items Schema
```
deferred:
  - id: BK-123
    reason: "pacing_limit|missing_evidence|oversized"
    required_action: "<describe>"
```

## Failure Block (Standard)
```
## Artifact:
(in-memory)
Status: FAILED
Missing: [new_signals|previous_value_map|agent_limits|human_context]
Deferred: [...]
Escalations: [MISSING_SIGNALS|PACING_VIOLATION|...]
```

## Success Block (Standard)
```
## Artifact:
.opencode/output/reprioritize/<timestamp_hash>.md
Status: SUCCESS
Missing: []
Deferred: [ ... ]
Escalations: [ ... ]
```

## Determinism Rules
- Weight constants fixed unless version incremented.
- Hash each row over (id,title,value_score,components,version).
- Identical inputs (signals set + previous map) → identical ordering & hashes.

## Escalation Conditions
| Escalation | Condition |
|------------|-----------|
| EXCESSIVE_CHURN | top10_reorder_pct > configured threshold |
| MISSING_SIGNALS | reason != scheduled AND new_signals empty |
| PACING_VIOLATION | new categories > max_new_concepts_per_iteration |

## Example Invocation
```
/reprioritize reason=new_signals new_signals=signals_2025-09-01
```

## Example Ranked Backlog (Abbrev)
```
| rank | id     | title                       | value_score | impact | adoption | friction | strategic | complexity | risk | sources        |
| 1    | BK-012 | Accelerate booking form UX  | 87.4        | 9      | 8        | 9        | 8         | 3          | 2    | LUA-2029,QRA-11|
| 2    | BK-018 | Visit note speed improvements | 75.1      | 8      | 7        | 8        | 7         | 4          | 3    | LUA-2031       |
```

## Non-Goals
- Implementation detail design
- Architecture changes (SYRA)
- Test planning (QRA)
- Simulation generation (LUA)

End of /reprioritize specification.
