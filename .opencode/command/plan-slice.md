---
name: /plan-slice
description: Decompose the current top-ranked backlog item into deterministic vertical slices within pacing & chunk constraints.
agent: maka
mode: analysis
version: 0.1.0
inputs:
  required:
    - backlog_item_id
  optional:
    - target_user_action_time
    - complexity_hint
    - dependencies
constraints:
  - Enforce preferred_chunk_size (from agent_limits)
  - Enforce function_size_lines (from agent_limits)
  - Introduce ≤ max_new_concepts_per_iteration new conceptual categories
  - Defer overflow slices with explicit Deferred list
outputs:
  - slice_matrix
  - pacing_report
  - deferred_items
escalation_triggers:
  - PACING_VIOLATION
  - CHUNK_OVERFLOW
  - MISSING_BACKLOG_ITEM
success_criteria:
  - Each slice spans DB → UI (vertical)
  - Each slice has acceptance_criteria_ref compatible with VIVA/QRA formats
  - No slice exceeds function_size_lines estimate
---

# /plan-slice Command Specification

## Purpose
Produce a deterministic vertical slice decomposition for the highest priority backlog item (from VIVA) ensuring adherence to pacing, chunk size, and function size constraints before implementation.

## Required Preconditions
1. Ranked backlog available (most recent VIVA artifact).
2. `human_context` and `agent_limits` accessible (provides pacing thresholds).
3. No active SECURITY_CRITICAL escalation from SYRA blocking the item.
4. Backlog item not already decomposed (idempotency check via hash if available).

## Procedure
1. Validate presence of backlog_item_id in ranked backlog.
2. Fetch constraints: preferred_chunk_size, function_size_lines, max_new_concepts_per_iteration.
3. Identify conceptual domains in item; if new domains exceed pacing limit → defer excess.
4. Draft initial slice candidates (target: smallest vertical increments achieving user-observable value).
5. For each candidate:
   - Estimate complexity (S/M/L) and line_span_estimate.
   - Attach acceptance_criteria_ref (linking to VIVA criteria or define minimal if absent).
   - Map test hooks (unit, integration, UX metric).
6. Prune or split slices violating constraints.
7. Emit slice_matrix + pacing_report + deferred_items.

## Slice Matrix Schema
| slice_id | title | goal | value_link (backlog_item_id) | line_span_est | complexity | acceptance_criteria_ref | risk_flag | dependencies |
|----------|-------|------|------------------------------|---------------|-----------|-------------------------|-----------|--------------|

## Pacing Report Schema
```
pacing_report:
  new_concepts_introduced: int
  max_allowed: int
  status: OK|VIOLATION
  deferred_concepts: [ ... ]
```

## Deferred Items Schema
```
deferred:
  - concept: string
    reason: "pacing_limit" | "chunk_overflow"
    recommended_phase: phase_number
```

## Failure Block (Standard)
```
## Artifact:
(in-memory)
Status: FAILED
Missing: [backlog_item_id|ranked_backlog|human_context|agent_limits]
Deferred: [...]
Escalations: [PACING_VIOLATION|CHUNK_OVERFLOW|...]
```

## Success Block (Standard)
```
## Artifact:
.opencode/output/plan-slice/<backlog_item_id>.md
Status: SUCCESS
Missing: []
Deferred: [ ... ]        # if any
Escalations: [ ... ]     # if any
```

## Determinism Rules
- Ordering of slices sorted by (user_value_impact DESC, complexity ASC, risk_flag ASC, title lexicographically).
- Hash each slice spec (stable JSON canonicalization) to detect drift.
- Identical inputs → identical slice_matrix & ordering.

## Acceptance Criteria Template
```
acceptance_criteria:
  primary_user_action: "<action>"
  target_time_seconds: n
  data_persistence: true|false
  localization_spanish: true
  accessibility_min: "WCAG-A"
  error_states_handled: [list]
```

## Example Invocation (Human)
```
/plan-slice backlog_item_id=BK-012 target_user_action_time=30s
```

## Example Output (Abbrev)
```
| slice_id | title                 | goal                      | line_span_est | complexity | acceptance_criteria_ref |
| S1       | Create patient form   | Basic create path         | 40            | S          | AC-BK-012-1             |
| S2       | List patients table   | View existing patients    | 30            | S          | AC-BK-012-2             |
| S3       | Edit patient record   | Modify core demographics  | 45            | M          | AC-BK-012-3             |
```

## Escalation Handling
- If PACING_VIOLATION: emit deferred concepts, do not forcibly continue.
- If CHUNK_OVERFLOW: split offending slice; if still > preferred_chunk_size mark as deferred.

## Non-Goals
- Implementation code
- Architecture changes (delegate to SYRA)
- UX validation (delegate to QRA & LUA feedback loop)

End of /plan-slice specification.
