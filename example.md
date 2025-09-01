# opencode Quick Start Example (Simplified)

Practical, command-first guide to start a new project session, verify agents & commands loaded, seed work, and run the core improvement loop.

---

## 1. One-Time Minimal Scaffold

```
opencode/
  opencode.json
  PROJECT_OBJECTIVE.md
  config_explain.md
  example.md
  .opencode/
    AGENTS.md
    agent/
      viva.md syra.md maka.md qra.md lua.md
    command/
      plan-slice.md arch-drift.md quality-pass.md simulate-users.md reprioritize.md
```

(Already present in this repo.)

---

## 2. Start Session & Verify Load

In opencode console:

```
# Show raw config (optional)
!cat opencode/opencode.json

# List registered instructions (should include)
!grep -n "PROJECT_OBJECTIVE" opencode/opencode.json

# (If framework provides) list agents / commands
/agents
/commands
```

Expected agents: viva, syra, maka, qra, lua, plus plan & build.  
Expected commands: /reprioritize /plan-slice /quality-pass /arch-drift /simulate-users

If something missing:
- Check paths in `opencode.json` (instructions, commands.directory)
- Ensure filenames match agent ids (e.g. `viva.md`).

---

## 3. Initialize Backlog (Manual Seed)

Create (or open) a backlog doc and add a few starter items with evidence references:

```
| id     | title                         | source                 | phase |
| BK-001 | Basic patient CRUD            | user_quote:DR-01       | 0     |
| BK-002 | Appointment booking core      | friction_log:LUA-INIT  | 0     |
| BK-003 | Payment mark paid/unpaid      | interview:RCPT-02      | 1     |
```

---

## 4. First Commands (Hour 0 → Value Path)

```
# 1. Attempt initial ranking (may fail if no signals yet)
 /reprioritize reason=scheduled

# 2. Decompose top item (replace with real backlog id)
 /plan-slice backlog_item_id=BK-001

# 3. Plan quality gate for earliest slice
 /quality-pass scope=slice:BK-001 slice_ids=BK-001

# 4. (If any code exists) architecture sanity
 /arch-drift scope=repo

# 5. Simulate baseline user flows (core clinic actions)
 /simulate-users scope=core max_scenarios=6

# 6. Re-rank with new signals
 /reprioritize reason=new_signals new_signals=LUA-INIT
```

Proceed to implement just the FIRST smallest slice from `/plan-slice` output.

---

## 5. Core Daily Loop (Minimal)

Morning:
```
/simulate-users scope=core
/reprioritize reason=new_signals new_signals=<SIM_ID>
```

Build Window:
```
/plan-slice backlog_item_id=<TOP_ID>
# Implement 1 slice
/quality-pass scope=slice:<SLICE_PARENT_ID> slice_ids=<SLICE_PARENT_ID>
```

Stability & Governance:
```
/arch-drift scope=repo
/reprioritize reason=scheduled
```

---

## 6. Command Purpose Cheat Sheet

| Command | When to Trigger | Output Focus |
|---------|-----------------|--------------|
| /reprioritize | New friction/adoption signals OR scheduled | Ranked backlog + churn report |
| /plan-slice | Before implementing a backlog item | slice_matrix + pacing report |
| /quality-pass | Pre-implementation (planning) or pre-deploy | quality_matrix + accessibility_report |
| /arch-drift | After structural change or routine check | drift_matrix + remediation_actions |
| /simulate-users | Daily baseline & post-change validation | friction_report + adoption_delta |

---

## 7. Minimal Artifact Expectations

After a healthy first loop you should have:
- `slice_matrix` for at least one backlog item.
- A quality matrix referencing acceptance criteria.
- Simulation friction/adoption delta driving reprioritization.
- (Optional) Drift matrix (often empty early if no infra creep).

---

## 8. Determinism Quick Audit

After running the same command twice with unchanged inputs:
```
/plan-slice backlog_item_id=BK-001
/plan-slice backlog_item_id=BK-001
```
Hashes or ordering of slices should remain identical. If not:
- Check that you didn’t manually alter inputs.
- Ensure no new backlog categories were introduced in-between.

---

## 9. First Hour Transcript (Reference)

```
> /reprioritize reason=scheduled
Status: FAILED (Missing signals)

> /plan-slice backlog_item_id=BK-001
Status: SUCCESS (S1..S3)

> /quality-pass scope=slice:BK-001 slice_ids=BK-001
Status: SUCCESS

(Implement S1)

> /simulate-users scope=core max_scenarios=5
Status: SUCCESS (Friction: booking_time_high)

> /reprioritize reason=new_signals new_signals=LUA-INIT
Status: SUCCESS (BK-002 moves to rank 1)
```

---

## 10. Escalation Handling (Fast Rules)

| Escalation | Gate Action |
|------------|-------------|
| SECURITY_CRITICAL | Remediate before new slice planning |
| ACCESSIBILITY_CRITICAL | Fix before deploy |
| EXCESSIVE_CHURN | Freeze new categories; stabilize backlog |
| PACING_VIOLATION | Defer excess concepts; do not force continuation |
| ADOPTION_COLLAPSE | Immediate `/reprioritize` + targeted slice |

---

## 11. Lightweight Decision Log Format

Add to `DECISIONS_LOG.md` (optional):
```
DEC-2025-09-01-01 :: source=LUA-INIT → change="Accelerate BK-002" → expected_delta="booking_time -15%"
```

---

## 12. Fast Extensions (Pick ONE Later)

| Extension | Command Idea |
|-----------|--------------|
| Metrics MCP | /metrics-digest |
| Security Scan MCP | /security-scan |
| Limits Centralization | Add agent_limits.json |
| Artifact Hash Audit | /integrity-scan |

---

## 13. Success Criteria (Early Phase)

- One completed vertical slice deployed.
- Simulation friction metric decreasing (booking flow time trending down).
- No PREMATURE_SCALE drift flags.
- No pacing violations unresolved.

Stay disciplined: only run commands that advance evidence-backed value.
