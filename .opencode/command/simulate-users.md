---
name: /simulate-users
description: Run (or plan) a deterministic batch of doctor + receptionist workflow simulations to produce friction/adoption deltas and scenario coverage report.
agent: lua
mode: analysis
version: 0.1.0
inputs:
  required:
    - scope
  optional:
    - target_actions
    - recent_features
    - adoption_baseline
    - max_scenarios
constraints:
  - Enforce scenario_batch_rules (agent_limits)
  - Seed all stochastic elements (seed = hash(scope + latest_backlog_rev))
  - Do not exceed max_new_concepts_per_iteration (new workflow categories)
  - Use only real or previously validated synthetic patient archetypes
outputs:
  - simulation_matrix
  - friction_report
  - adoption_delta
  - coverage_summary
escalation_triggers:
  - ADOPTION_COLLAPSE
  - FRICTION_SPIKE
  - PACING_VIOLATION
success_criteria:
  - Each scenario traces to at least one backlog item or acceptance_criteria_ref
  - Friction metrics quantified (time_seconds, click_count, interruption_penalty)
  - Adoption deltas reference prior baseline snapshot id
---

# /simulate-users Command Specification

## Purpose
Produce a reproducible set of doctor + receptionist simulated workflows to surface new or persisting friction points, update adoption metrics, and feed VIVA (reprioritization), QRA (quality planning), and MAKA (slice refinement).

## Required Preconditions
1. Latest ranked backlog (for mapping scenario relevance).
2. Prior adoption baseline snapshot (if exists) for delta computation.
3. `agent_limits` (scenario_batch_rules, pacing thresholds).
4. Phase context (from `PROJECT_OBJECTIVE.md`) to filter out future-phase scenarios.
5. Optional: recent_features list to bias scenario selection.

## Scope Interpretation
- scope = `core` → Run canonical baseline daily scenarios.
- scope = `feature:ID` → Focus scenarios linked to backlog item or slice set.
- scope = `regression` → Re-run last scenario set for drift detection.

## Procedure
1. Resolve seed = SHA256(scope + latest_backlog_rev + (recent_features||"none")).
2. Gather candidate scenario templates (doctor / receptionist archetypes).
3. Rank candidates by (recent_feature_relevance DESC, friction_history DESC, adoption_gap DESC).
4. Enforce scenario_batch_rules:
   - Max scenarios per batch
   - Max parallel doctor vs receptionist ratio
5. Trim or defer scenarios exceeding pacing (new conceptual categories).
6. Execute (or plan if dry-run) each scenario step:
   - Record action_time_seconds, click_count, context_switches, errors, interruptions.
7. Compute friction_score = weighted_sum(time_over_target, click_over_target, interruption_penalty).
8. Aggregate adoption metrics (usage_frequency, completion_rate).
9. Produce friction_report (sorted by severity).
10. Emit adoption_delta comparing previous baseline (if available).

## Simulation Matrix Schema
| scenario_id | role | workflow | target_action | time_seconds | click_count | interruptions | errors | friction_score | linked_items | batch_id |

## Friction Report Schema
| rank | scenario_id | friction_score | primary_factor | delta_vs_prev | severity | recommendation |

Severity mapping deterministic (score thresholds).

## Adoption Delta Schema
| action | prev_frequency | new_frequency | delta | adoption_status |
adoption_status ∈ {IMPROVED, REGRESSED, STABLE}

## Coverage Summary Schema
```
coverage:
  total_scenarios: n
  doctor: n1
  receptionist: n2
  unique_actions: k
  action_coverage_pct: X
  deferred_scenarios:
    - scenario: "<name>"
      reason: "pacing_limit|batch_limit"
```

## Failure Block (Standard)
```
## Artifact:
(in-memory)
Status: FAILED
Missing: [ranked_backlog|agent_limits|scope]
Deferred: [...]
Escalations: [PACING_VIOLATION|...]
```

## Success Block (Standard)
```
## Artifact:
.opencode/output/simulate-users/<scope_hash>.md
Status: SUCCESS
Missing: []
Deferred: [ ... ]
Escalations: [ ... ]
```

## Determinism Rules
- Scenario selection ordering derived from stable sort key tuple:
  (recent_feature_relevance DESC, friction_history DESC, adoption_gap DESC, scenario_name ASC).
- Seed governs any tie-break simulation randomness.
- Identical inputs and seed → identical metrics (except when real telemetry appended; then mark as NON_PURE).

## Escalation Conditions
| Escalation | Condition |
|------------|-----------|
| ADOPTION_COLLAPSE | adoption_delta shows > X% regression on core action (configurable threshold) |
| FRICTION_SPIKE | friction_score increase > Y absolute or Z% vs last run |
| PACING_VIOLATION | New conceptual categories > max_new_concepts_per_iteration |

## Example Invocation
```
/simulate-users scope=core max_scenarios=12
```

## Example Simulation Matrix (Abbrev)
```
| scenario_id | role | workflow            | target_action      | time_seconds | click_count | interruptions | friction_score | linked_items |
| SC-01       | receptionist | book_appointment_peak | schedule_patient | 28           | 9           | 1             | 0.12          | BK-012       |
| SC-02       | doctor       | add_visit_note        | record_visit     | 95           | 15          | 0             | 0.30          | BK-018       |
```

## Non-Goals
- Creating new feature definitions
- Architecture or infra recommendations (SYRA domain)
- Quality gate design (QRA domain, though outputs feed it)

End of /simulate-users specification.
