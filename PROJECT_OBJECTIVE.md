# Lumina Health CRM – Project Objective (Condensed Operational Core)

## 1. One-Line Product Thesis
Spanish-first lightweight CRM replacing fragmented WhatsApp + Excel workflows for solo and small Chilean clinics, delivering scheduling, patient records, and visit payment tracking fast enough to secure 3 paying clinics inside 30 days and reach $500–$1000 MRR within 90 days.

## 2. Core Objective (90-Day Horizon)
1. Secure 3 paying Villarrica solo practitioners by Day 30 (Phase 1 close).  
2. Validate daily clinical scheduling + basic patient record + payment marking loop with sub-60s admin overhead per appointment.  
3. Reach operational stability to support 25 customers with minimal rework by Day 90 (scalable foundation: upgrade path not activated early).  

## 3. Phase Progression (Activation Gates)
| Phase | Window | Ship Only | Exit Gate (Advance When) |
|-------|--------|-----------|---------------------------|
| 0 Pre-MVP | Days 1–7 | Patient CRUD + basic calendar | 3 real demos + specific workflow feedback collected |
| 1 Revenue MVP | Days 8–30 | Scheduling, minimal patient records, payment marking, manual WhatsApp reminder template | 3 paying clinics ($90 MRR) |
| 2 Growth | Days 31–60 | Add only features users explicitly request (reminders automation, basic reports) | $300 MRR + validated daily usage |
| 3 Scale Bootstrap | Days 61–90 | Compliance & multi-doctor enablers (only after traction) | $500 MRR + churn & adoption stable |

No development beyond current phase even if “almost ready” (strict gating).

## 4. Success Metrics (Tracked Weekly)
- Traction: Demo → Paid conversion ≥ 30%.  
- Engagement: 80% of booked appointments marked with final status same day.  
- Adoption: Daily agenda view used ≥ 5 days/week per clinic.  
- Efficiency: Receptionist booking flow < 30s median, payment mark < 10s.  
- Revenue: $90 MRR (Day 30), $300 (Day 60), $500–$1000 (Day 90).  
- Scope Discipline: < 10% of shipped code replaced within 30 days (avoids churn from over-engineering).  

## 5. Operating Constraints (Enforced by Agents)
| Constraint Type | Rule |
|-----------------|------|
| Over-Engineering Guard | No feature merges without mapped user pain + phase alignment |
| Architecture Deferral | No React / PostgreSQL / multi-tenant before Gate 2 |
| Frontend Simplicity | Bootstrap + Jinja only until Growth Phase |
| Security Minimal Set | HTTPS, hashed passwords, ORM = Phase 1 ceiling |
| Time Boxing | 2h max per work block; ship partial not perfect |
| Feedback Loop Cadence | At least one real user interaction daily (demo, feedback, usage observation) |
| Manual Before Automation | WhatsApp reminders: manual template copy until automation is paid request |
| Spanish-Only Scope | No bilingual logic until ≥ 10 paying clinics |

## 6. Non-Goals (Until After Scale Gate)
- Multi-clinic hierarchy & granular permissions.
- Full audit logging & encryption at rest.
- Automated WhatsApp messaging platform integration (only manual templated messaging first).
- Advanced analytics dashboards.
- Mobile app (PWA only if absolutely demanded after Phase 2).
- Sophisticated billing / Stripe integration (manual transfer accepted).

## 7. Guiding Principles
1. Ship over sophistication.  
2. Manual validates value before automation invests time.  
3. Local clinician workflow reality overrides “ideal architecture”.  
4. Deterministic improvement: each cycle must produce measurable delta (speed, adoption, MRR, friction removal).  
5. Health & pacing guard sustainability → consistent output > sporadic overexertion.

## 8. Agent Alignment (Operational Loop)
```
LUA → VIVA → MAKA → SYRA → QRA → LUA (continuous)
```
- VIVA: Re-scores backlog daily from LUA friction + adoption signals.  
- MAKA: Implements smallest viable vertical slice (DB → UI) with Spanish-first naming.  
- SYRA: Approves only security/performance-critical infra changes; rejects premature scaling.  
- QRA: Validates usability (click count, time-to-complete) + accessibility & Spanish fidelity.  
- LUA: Simulates real receptionist + doctor flows; produces friction deltas & adoption metrics.

## 9. Validation & Quality Gates
Each shipped slice must include:
- Defined target user action time (booking, payment marking, note capture).  
- LUA friction report after simulation cycle.  
- QRA acceptance block: accessibility pass, click count, performance response check.  
- SYRA sign-off if change introduces schema or deployment footprint modifications.

## 10. Risk Mitigation Mapping
| Risk | Countermeasure |
|------|----------------|
| Over-engineering backend | Enforce Phase 0–1 tech ceiling |
| Feature creep | VIVA rejects backlog items w/o LUA friction evidence |
| No customer uptake | Daily in-person / direct contact schedule tracked |
| Health energy dips | Time-box & allow partial slice acceptance |
| UI complexity slowdown | Template reuse + Bootstrap canonical components |

## 11. Immediate Tactical Focus (Active Sprint Objective)
Deliver Phase 0 assets: patient CRUD + basic appointment calendar + deploy link shown to 3 practitioners → collect structured feedback (pain categories: booking time, info completeness, navigation friction).

## 12. Escalation Rules
- Any attempt to add infra not in Phase list → SYRA block + backlog note.  
- Feature lacking explicit LUA pain reference ID → VIVA auto-reject.  
- Slice exceeding agreed function size or complexity → MAKA decomposes before continuing.  

## 13. Deterministic Traceability
All decisions link to: `DECISION_ID :: {source:pain|metric|user_quote} → {change} → {expected_metric_delta}` appended to a log (see AGENTS.md governance).

## 14. Reference Documents
- Detailed PRD: `projects/med_crm/enhanced_PRD_medicalCRM.md`
- Agent Rules & Governance: `.opencode/AGENTS.md`
- Evolved Agent Specs: `khujta/010_evolved_agents/`
- (Planned) Commands: `.opencode/command/*.md`
- (Planned) MCP Config: `opencode.json` (`mcp` section)

## 15. Change Protocol
Any modification to objectives requires:
1. LUA / real user new friction or metric shift.  
2. VIVA value reassessment.  
3. Logged decision entry.  
4. Propagated constraint review (SYRA + QRA).  

---

Last Sync: (placeholder - update via automation)  
Status: ACTIVE (Phase 0 / 1 transition pending Gate 0 exit)
