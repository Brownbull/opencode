# Project Agents & Operational Rules

This repository uses an evolved multi-agent pattern (VIVA, SYRA, MAKA, QRA, LUA) adapted for opencode subagents. These rules guide agent collaboration, pacing, and tool usage.

## 1. High-Level Objective
Deliver a lean, Spanish‑first healthcare CRM that replaces fragmented WhatsApp + Excel workflows for small Chilean clinics, targeting first paid clinics rapidly (see `PROJECT_OBJECTIVE.md` for condensed PRD). Priority: ship revenue-capable slices fast (value > perfection).

## 2. Agent Ensemble (Subagents)
| Agent | Purpose (Concise) | Primary Artifact / Output Mode |
|-------|-------------------|--------------------------------|
| viva  | Deterministic backlog ranking & acceptance criteria synthesis | Ranked value map / prioritization summary |
| syra  | Architecture & infra drift + security & performance alignment | Architecture alignment recommendations |
| maka  | Vertical slice decomposition & implementation planning (TDD-first) | Implementation plan & slice matrix |
| qra   | Reality + quality validation (accessibility, usability, performance) | Quality review & defect classification |
| lua   | Workflow simulation (doctor/receptionist) & friction/adoption signaling | Adoption & friction deltas summary |

All five subagent prompt bases live in `.opencode/agent/*.md`.

## 3. Cognitive & Decomposition Constraints
Sourced from `human_context` + `agent_limits` (referenced externally):
- max_new_concepts_per_iteration: Limit concept/category introductions per cycle.
- preferred_chunk_size: Maximum slice / batch size before enforced decomposition.
- concepts_before_break: Insert pacing break after this many distinct new concepts.
- function_size_lines (MAKA), architecture_batch_units (SYRA), validation_batch_units (QRA), scenario_batch_rules (LUA) enforced.
Agents MUST explicitly state when deferring concepts or chunks.

## 4. Determinism & Reproducibility
- Identical structured inputs → identical ordering, scores, and hashes.
- Random simulation (LUA) must be seeded.
- Scoring formulas (VIVA, SYRA, QRA) stable unless versioned.
- Each agent includes failure codes rather than freeform error prose.

## 5. Escalation Triggers (Abbrev)
| Trigger | Origin | Escalate To | Action |
|---------|--------|-------------|--------|
| EXCESSIVE_CHURN | VIVA | Human + SYRA | Review prioritization weights |
| SECURITY_CRITICAL | SYRA | VIVA + MAKA | Block affected work; prioritize remediation |
| ACCESSIBILITY_CRITICAL | QRA | MAKA + VIVA | Remediate before deploy |
| ADOPTION_COLLAPSE | LUA | VIVA + MAKA | Re-evaluate UX & value assumption |
| PACING_VIOLATION (Repeated) | Any | Human Oversight | Recalibrate thresholds |

## 6. Interaction Loop (Core Flow)
```
LUA → VIVA → MAKA → SYRA → QRA → LUA  (continuous feedback)
```
Side channels: VIVA ↔ SYRA (cost/compliance), QRA ↔ MAKA (defects), LUA ↔ QRA (scenario refinement).

## 7. Required Agent Behaviors in opencode
- Subagents NEVER write or edit code unless explicitly granted (current permissions: read/analysis only).
- Primary `build` agent can implement; `plan` agent performs safe analysis.
- If a task risks violating pacing or size constraints, agent must:
  1. Stop expansion
  2. Emit deferred list
  3. Provide next-step recommendation

## 8. Instruction Loading Strategy
When referencing external context (e.g., extended guidelines, data models):
- Lazy load only when task demands.
- Avoid preloading large, unrelated files.
- Use deterministic summaries (tables / bullet matrices) over narrative.

## 9. File Reference Policy
When a specification references another file (e.g., `PROJECT_OBJECTIVE.md`, evolved agent specs):
1. Read only once per active reasoning chain unless updated.
2. Summarize using stable headings.
3. Re-derive hashes for change detection if needed.

## 10. Failure Mode Standardization
Agents must return structured status blocks:
```
## Artifact:
<path or in-memory>
Status: SUCCESS|FAILED
Missing: [...]
Deferred: [...]
Escalations: [...]
```
No extra commentary outside defined fields.

## 11. Quality & Pacing Enforcement Matrix (Abbrev)
| Aspect | Primary Agent | Secondary | Enforcement Point |
|--------|---------------|----------|-------------------|
| Concept pacing | VIVA | LUA/QRA | Ranking / category intro |
| Chunk size | MAKA | SYRA/QRA/LUA | Slice planning / batches |
| Architecture batch size | SYRA | MAKA | Evolution action segmentation |
| Validation batch | QRA | LUA | Coverage execution |
| Scenario batch size | LUA | QRA | Simulation selection |
| Function size | MAKA | QRA | Slice decomposition |

## 12. Command Layer (Planned)
Custom commands to be added under `.opencode/command/` (see forthcoming files):
- `/plan-slice` – Decompose top backlog item (MAKA perspective).
- `/arch-drift` – Summarize current architectural drift (SYRA).
- `/quality-pass` – Run quality validation planning (QRA).
- `/simulate-users` – Propose next scenario batch (LUA).
- `/reprioritize` – Re-rank backlog with new signals (VIVA).

## 13. MCP Integration Baseline
Planned (placeholders in `opencode.json`):
- Local: performance metrics collector
- Remote: security scan aggregator
Agents should only enable specific MCP tools they need (principle of least privilege).

## 14. Usage Guidance for Contributors
1. Start in `plan` agent for analysis.
2. Use subagents via `@viva`, `@maka`, etc. for domain-specific structuring.
3. Execute implementation only after:
   - VIVA ranking stable (no churn > threshold)
   - SYRA confirms no SECURITY_CRITICAL
   - QRA baseline gates green
4. After each meaningful change, run `/arch-drift` & `/quality-pass` to detect regressions early.

## 15. Non-Goals
- No premature multi-tenant abstractions.
- No speculative architectures beyond immediate constraints.
- Avoid monolithic “mega features”; require vertical slice decomposition.

## 16. Update Protocol
Any change to scoring formulas, pacing thresholds, or batch rules:
1. Update relevant agent markdown.
2. Reflect matrix adjustments here.
3. Add/update command if workflow step changes.
4. Commit with `chore(agents): update pacing/scoring` message.

## 17. Quick Reference Hash Checklist
- [ ] Concept pacing enforced
- [ ] Chunk size compliance
- [ ] Batch decomposition present
- [ ] Deterministic ordering preserved
- [ ] Escalations explicitly surfaced
- [ ] No uncontrolled write intentions by subagents

---

This `AGENTS.md` is authoritative for opencode session context. Keep concise; expand deep domain details in separate docs referenced on-demand.
