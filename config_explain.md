# opencode Configuration Explained

This document explains every file and directory inside the `opencode` project scaffold, how the opencode framework consumes them, and how they interrelate to enable a deterministic multi‑agent workflow.

---

## 1. Root Files

### `opencode.json`
Central configuration loaded by the opencode runtime.
- `permission`: Default global tool permission policy (edit / bash / webfetch).
- `agent`: Declares primary agents (`build`, `plan`) and subagents (`viva`, `syra`, `maka`, `qra`, `lua`):
  - Each subagent has: `mode`, `model`, `temperature`, allowed `tools`, and `permission` overrides.
  - Markdown prompt bodies live separately under `.opencode/agent/*.md` (single source of truth for role behavior). JSON only adds operational metadata (model, temp, tool gating).
- `instructions`: Ordered list of markdown files to automatically inject as shared instruction context (`.opencode/AGENTS.md`, `PROJECT_OBJECTIVE.md`).
- `commands`: Points to directory containing command specification markdowns (`.opencode/command`).
- `mcp`: Declares (optional) Model Context Protocol servers:
  - `servers`: Named server definitions (transport, command/url, args, enabled flag, description).
  - `policies`: Allow / deny lists (currently placeholders).
- `notes`: Human-readable reminder of separation of concerns.

Consumption order during session spin‑up:
1. Load `opencode.json`.
2. Resolve `instructions` → concatenate / index.
3. Register agents (JSON metadata + respective `.opencode/agent/*.md` bodies).
4. Register commands from declared directory.
5. Initialize (disabled) MCP server definitions for later activation.

### `PROJECT_OBJECTIVE.md`
Condensed operational objective & phase gating.
- Provides phase advancement conditions (Gates 0→3).
- Enforced indirectly by subagents:
  - `viva` uses phase gates to reject backlog items out of scope.
  - `syra` blocks premature infra changes referencing gate constraints.
  - `maka` limits decomposition to immediate phase value slices.
  - `qra` validates performance & usability within defined scope.
  - `lua` filters simulation scenarios by active phase.
Added to `instructions` → always in shared context.

---

## 2. Governance & Agent Definition Layer

### `.opencode/AGENTS.md`
Global operational rules:
- Interaction loop definition (LUA → VIVA → MAKA → SYRA → QRA → LUA).
- Determinism, pacing & decomposition constraints; escalation triggers.
- Standard failure/success status block schema.
- Command registry intentions (ensures commands align with governance).
- Acts as canonical enforcement matrix (pacing, batch, chunk size, function size).
Referenced in `instructions` so every agent inherits these baseline rules.

### `.opencode/agent/` (Directory)
Holds per‑subagent specialized behavior markdowns (pure prompts):
- `viva.md`: Backlog ranking & acceptance criteria synthesizer (deterministic scoring).
- `syra.md`: Architecture / infra drift & security validation advisor (phase guarding).
- `maka.md`: Slice decomposition & implementation planning (no writes).
- `qra.md`: Quality, accessibility, and performance validation planner.
- `lua.md`: User workflow simulation & friction/adoption signal synthesis.

How opencode uses them:
1. On session initialization, each file is mapped to the agent id matching its filename.
2. The runtime merges:
   - Core text (behavior, constraints) from markdown.
   - Operational metadata from `opencode.json` (model, tools, permissions).
3. When invoked (`@viva`, `/reprioritize`, etc.), the framework feeds these prompt definitions plus the shared `instructions`.

Separation rationale:
- Markdown remains model/vendor agnostic.
- JSON adjusts runtime execution without altering behavioral contract.

---

## 3. Command Layer

Directory: `.opencode/command/`
Each file = a command registration spec. The framework scans this directory (set in `opencode.json.commands.directory`) and exposes commands as slash actions.

| File | Agent | Purpose (Concise) |
|------|-------|-------------------|
| `plan-slice.md` | maka | Deterministic vertical slice decomposition (DB→UI) respecting pacing & chunk constraints. |
| `arch-drift.md` | syra | Detect architecture/security drift & premature scaling vs phase gates. |
| `quality-pass.md` | qra | Plan validation batches (usability, accessibility, performance, localization). |
| `simulate-users.md` | lua | Run / plan seeded workflow simulations producing friction & adoption deltas. |
| `reprioritize.md` | viva | Re-rank backlog using new signals & compute churn metrics. |

Common structure fields:
- Front‑matter: `name`, `agent`, `mode`, `inputs`, `constraints`, `outputs`, `escalation_triggers`, `success_criteria`.
- Body: Procedure, schemas, determinism rules, example invocation & outputs, failure/success status blocks.

Runtime behavior:
1. User (or automation) issues `/command param=value`.
2. Framework resolves associated subagent, injects:
   - Command spec
   - Shared instructions
   - Relevant contextual artifacts (if present)
3. Agent returns structured artifact(s) & status block.

Determinism goal:
- Each command spec defines ordering, hashing, or weighting formulas.
- Ensures reproducible outcomes for same input set (supports auditing & drift detection).

---

## 4. MCP (Model Context Protocol) Integration

Defined in `opencode.json.mcp.servers` (currently placeholders):
- `local-metrics`: Potential stdio Python server (disabled) for real runtime metrics (latency, resource usage) feeding `syra` / `qra`.
- `remote-security-scan`: HTTP endpoint placeholder for security posture ingestion.

Activation flow:
1. Set `enabled: true`.
2. Ensure command / tool gating updates agent permissions to allow new tool usage.
3. Optionally add a command spec leveraging MCP results (e.g., `/security-scan`).

---

## 5. Cross‑File Relationships & Data Flow

```
PROJECT_OBJECTIVE.md ─┐
                       │
.opencode/AGENTS.md ───┼─► (Shared Instruction Layer) ► All subagents
                       │
.opencode/agent/*.md ──┘     (Behavioral specializations)

Commands (.opencode/command/*.md) ─► Trigger structured agent executions

Outputs (planned):
.opencode/output/<command>/<artifact>.md  (referenced in status blocks)

MCP servers (future) ─► Provide external signal inputs (metrics/security)
Signals (lua/qra/syra) ─► feed viva (/reprioritize) ─► influences maka (/plan-slice)
```

Escalation propagation examples:
- `syra` sets `SECURITY_CRITICAL` → blocks `/plan-slice` (MAKA) until resolved.
- `qra` `ACCESSIBILITY_CRITICAL` → prevents deployment slice acceptance.
- `lua` `ADOPTION_COLLAPSE` → forces `/reprioritize` early.

---

## 6. Determinism & Auditability Surfaces

| Mechanism | Files Involved | Purpose |
|-----------|----------------|---------|
| Weight & scoring formulas | `viva.md`, `reprioritize.md` | Stable backlog ranking |
| Hashing rules | Command specs | Drift detection of slice/test/scenario definitions |
| Pacing thresholds | `.opencode/AGENTS.md` + future `agent_limits` file (external) | Cognitive load governance |
| Phase gates | `PROJECT_OBJECTIVE.md` | Scope control / anti-scope creep |
| Standard status block | All command specs | Machine-readable orchestration & pipeline gating |

---

## 7. Extending the Configuration

| Extension | Steps |
|-----------|-------|
| Add new subagent | Create `.opencode/agent/<name>.md`, add config under `agent` in `opencode.json`, optionally add command spec. |
| Add command | Drop spec into `.opencode/command/`, ensure referenced agent exists. |
| Add instructions file | Author file → append path to `instructions` array (ordering matters for precedence). |
| Enable MCP server | Flip `enabled` to `true`, supply real `command` / `url`, adjust agent permissions if needed. |
| Introduce new constraint | Update `.opencode/AGENTS.md` + relevant agent / command specs; add enforcement to matrix. |

---

## 8. Why This Separation Matters

| Concern | Solved By |
|---------|-----------|
| Behavioral clarity | Dedicated agent markdowns |
| Operational tunability | `opencode.json` metadata |
| Deterministic orchestration | Command specs with schemas & ordering rules |
| Scope discipline | Phase gates in `PROJECT_OBJECTIVE.md` |
| Governance consistency | `.opencode/AGENTS.md` |
| Future observability & security | MCP server placeholders |

---

## 9. Quick Validation Checklist (Post-Change)

- [ ] `opencode.json` validates against schema.
- [ ] Each command spec has unique `name`.
- [ ] All agent ids in commands exist in `opencode.json.agent`.
- [ ] Instructions list includes required governance + objective docs.
- [ ] Phase gates unchanged or version-bumped (if semantics adjusted).
- [ ] Determinism sections present in new command/agent specs.
- [ ] No command escalations reference undefined codes.

---

## 10. Planned Future Additions (Optional)

| Candidate | Rationale |
|-----------|-----------|
| `agent_limits.json` | Centralize numeric pacing / chunk thresholds versionably. |
| `/security-scan` command | Consume MCP remote security server outputs. |
| `/metrics-digest` command | Aggregate performance metrics → feed `viva` strategic weighting. |
| Output archive rotation | Purge old artifacts, keep last N per command for audit. |

---

For implementation changes, always modify the narrowest file (single-responsibility) and rely on the layered load order to propagate behavior.
