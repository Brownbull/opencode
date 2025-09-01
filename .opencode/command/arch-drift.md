---
name: /arch-drift
description: Analyze current implementation or planned slices for architecture & infrastructure drift, security gaps, and premature scaling attempts.
agent: syra
mode: analysis
version: 0.1.0
inputs:
  required:
    - scope
  optional:
    - recent_commits
    - candidate_slices
    - deployment_changes
constraints:
  - No proposing tech outside current phase gates
  - Enforce architecture_batch_units (agent_limits)
  - Flag premature optimization (React, PostgreSQL, multi-tenant, complex auth)
  - Report only deterministic recommendations (stable ordering)
outputs:
  - drift_matrix
  - remediation_actions
  - blocked_changes
escalation_triggers:
  - SECURITY_CRITICAL
  - PREMATURE_SCALE
  - COMPLIANCE_RISK
success_criteria:
  - Each drift entry includes impact, severity, phase_alignment, remediation
  - No remediation exceeds preferred_chunk_size without decomposition
  - SECURITY_CRITICAL items sorted first
---

# /arch-drift Command Specification

## Purpose
Provide a deterministic snapshot of architecture & infra posture versus allowed Phase constraints (see `PROJECT_OBJECTIVE.md`), flagging drift, security gaps, and premature scaling proposals before they propagate.

## Required Preconditions
1. Phase context available (from `PROJECT_OBJECTIVE.md` gates).
2. `agent_limits` accessible for architecture_batch_units.
3. Provided `scope` clarifies target (e.g., "repo", "slice:BK-012", "deployment").
4. No missing mandatory security baseline (HTTPS, password hashing) unacknowledged.

## Procedure
1. Interpret `scope`:
   - repo → holistic scan (logical; may rely on metadata)
   - slice:* → evaluate slice plan for architectural integrity
   - deployment → validate infra alignment (resource shape, automation gating)
2. Collect candidate signals (recent_commits, candidate_slices, deployment_changes if provided).
3. Normalize into component categories: persistence, interface, security, observability, deployment, compliance, performance.
4. For each category:
   - Detect drift vs Phase baseline.
   - Assess severity (CRITICAL|HIGH|MED|LOW) with deterministic mapping.
   - Determine phase_alignment: ALLOWED|DEFER|BLOCK.
5. Generate remediation actions; split actions exceeding architecture_batch_units.
6. Sort output:
   - SECURITY_CRITICAL first
   - Then severity (CRITICAL→LOW)
   - Then phase_alignment (BLOCK→DEFER→ALLOWED)
   - Then category lexicographically.

## Drift Matrix Schema
| id | category | issue | severity | phase_alignment | risk_vector | remediation | est_batch_units | escalation |
|----|----------|-------|----------|-----------------|-------------|-------------|-----------------|------------|

## Remediation Action Schema (YAML)
```
remediation_actions:
  - id: DR-001
    linked_issue: <id>
    phase_gate_required: false
    decomposition: [step1, step2, ...]   # each ≤ architecture_batch_units
    expected_outcome: "<measurable>"
    risk_reduction: "<metric or qualitative>"
```

## Blocked Changes Schema
```
blocked_changes:
  - proposal: "<description>"
    reason: "PREMATURE_SCALE|SECURITY_CRITICAL|OUT_OF_PHASE"
    required_phase: <phase_number>
```

## Failure Block (Standard)
```
## Artifact:
(in-memory)
Status: FAILED
Missing: [scope|phase_context|agent_limits]
Deferred: [...]
Escalations: [SECURITY_CRITICAL|...]
```

## Success Block (Standard)
```
## Artifact:
.opencode/output/arch-drift/<scope_hash>.md
Status: SUCCESS
Missing: []
Deferred: [ ... ]
Escalations: [ ... ]
```

## Determinism Rules
- Severity mapping table versioned; identical inputs yield same ordering.
- Hash of (category, issue, risk_vector) forms stable id (DR-HASH).
- Batch decomposition stable given same est_batch_units.

## Severity Mapping (Deterministic)
| condition | severity |
|-----------|----------|
| Missing baseline security (HTTPS, hashing) | CRITICAL |
| Data exposure risk (PII without access guard) | CRITICAL |
| Premature infra (PostgreSQL before gate) | HIGH |
| Performance tuning pre-metric | LOW |
| Non-blocking observability gap | MED |

## Escalation Handling
- SECURITY_CRITICAL → escalate & block related slices until remediation plan accepted.
- PREMATURE_SCALE → mark DEFER, provide minimal justification.

## Non-Goals
- Implementing fixes
- Performance micro-optimizations absent metrics
- Recommending future-phase stack migrations prematurely

## Example Invocation
```
/arch-drift scope=repo recent_commits=last20
```

## Example Drift Matrix (Abbrev)
```
| id        | category | issue                         | severity | phase_alignment | risk_vector         | remediation                | est_batch_units | escalation |
| DR-a1b2c3 | security | Password hashing absent       | CRITICAL | BLOCK           | credential_compromise | Add bcrypt hashing (split) | 2               | SECURITY_CRITICAL |
| DR-d4e5f6 | data     | PostgreSQL proposal premature | HIGH     | DEFER           | premature_scale      | Defer to Phase 2           | 1               | PREMATURE_SCALE |
```

End of /arch-drift specification.
