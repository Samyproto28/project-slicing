---
name: project-slicing
description: Use when you have a complete PRD or product requirements document for an MVP or large feature set with multiple subsystems. Triggers on PRD decomposition, MVP phasing, project roadmap creation, or when writing-plans rejects a spec for covering multiple subsystems.
---

# Project Decomposition

## Quick start

| Mode | Input | Output | When |
|------|-------|--------|------|
| ROADMAP | Full PRD | Phased roadmap | First time with the PRD |
| SPEC | Approved roadmap | Individual phase spec | One per phase |

1. **Have a PRD?** → ROADMAP MODE: read PRD → interview → generate phased roadmap
2. **Have an approved roadmap?** → SPEC MODE: read roadmap + codebase → generate one phase spec
3. **Have a phase spec?** → pass to `writing-plans` for implementation

Input: **TXT, Markdown, HTML** only. No PDF. Edge cases: see [REFERENCE.md](REFERENCE.md).

**Do NOT use when:** PRD has 1-2 features (use `brainstorming` + `writing-plans`) or you already have a scoped single-subsystem spec.

---

## ROADMAP MODE

Paste this checklist into your response and update it as you complete each step:

```
ROADMAP Progress:
- [ ] Step 1: Read PRD + explore codebase
- [ ] Step 2: Refinement interview (one question at a time)
- [ ] Step 3: Generate roadmap (vertical slices, 3-5 stories/phase)
- [ ] Step 4: Save + get user approval
```

### 1. Read PRD + Explore Codebase

Read full PRD. If codebase exists, explore ONLY: root `ls` (one level), `package.json`, `glob: **/models/*, **/routes/*, **/api/*`. Greenfield? Skip.

### 2. Refinement Interview

Ask **exactly ONE question per message**, multiple choice when possible. Wait for answer before next. Only about what the PRD does not clarify:

- **Priorities:** must-have vs nice-to-have?
- **Scope:** "Billing — MVP-critical or post-launch?"
- **Ambiguities:** "Real-time — WebSocket, SSE, or polling?"
- **Technical risks:** "X technology — familiar or prefer alternative?"

**NEVER skip.** Every PRD has assumptions. NEVER make priority decisions without user. Features ONLY deferred if user confirms. **DO NOT batch.** ONE question, wait for answer, then next.

If user provides priority input (e.g., "CTO says X in Phase 1"), verify it doesn't break technical dependencies. If it does, report the conflict and ask the user to confirm or adjust. You still don't decide — you just surface the dependency issue.

### 3. Generate Roadmap

**Criteria:** Vertical slices (DB+API+UI) ordered by business value, respecting dependencies. Rules: 3-5 stories per phase (>5: subdivide, <2: merge). Each phase must deliver something functional (no pure-backend or pure-UI phases). Core features before complementary.

**DO NOT:** generate phase specs yet, invent features, create horizontal phases, make priority calls without user.

### 4. Save + Approval

Save to `docs/decomposition/roadmap/YYYY-MM-DD-<project>-roadmap.md`: executive summary, dependency graph, phase table (with status column — see [REFERENCE.md](REFERENCE.md) for lifecycle), deferred features, parallelizable phases.

**DO NOT proceed to SPEC MODE until user approves.**

---

## SPEC MODE

Paste this checklist into your response and update it as you complete each step:

```
SPEC Progress:
- [ ] Step 1: Context reading (strict scope)
- [ ] Step 2: Determine phase
- [ ] Step 3: Mini-interview
- [ ] Step 4: Generate phase spec
- [ ] Step 5: Save + handoff to writing-plans
```

### 1. Context Reading (STRICT)

Read in order: (1) roadmap, (2) previous phase specs, (3) codebase ONLY — root ls, `glob **/models/*, **/schemas/*, **/types/*, **/routes/*, **/api/*`, package.json. DO NOT read UI components, test files, or service implementations. Need something else? ASK first. **Violating reading rules wastes context and causes hallucinations.**

### 2. Determine Phase

User specifies → generate that phase. User says "next phase" → find first `pending` phase whose dependencies are all `completed`. Status lifecycle: pending → spec-generated → in-progress → completed (see [REFERENCE.md](REFERENCE.md)).

### 3. Mini-Interview

Ask: "Anything changed since last phase?" and "Roadmap says Phase N covers X and Y — still accurate?" These two can go in one message. Any additional questions (scope changes, cross-phase drift, ambiguities, feature clarifications) follow the same rule as ROADMAP MODE: **ONE question per message, wait for answer, then next.** A "follow-up" to a base question is still an additional question — it gets its own message. If a `brainstorming` spec exists for this phase, use as base.

### 4. Generate Phase Spec

For each feature: **components** (name + one-line responsibility), **data models** (entities, fields, relationships — NO queries/logic), **API contracts** (method, path, request, response — NO implementation), **user flows**, **acceptance criteria**, **dependencies on previous phases**. Max 300 lines. Format and example: see [REFERENCE.md](REFERENCE.md).

DO NOT invent features. If ambiguous, ASK. If codebase contradicts roadmap, REPORT.

### 5. Save + Handoff

Save to `docs/decomposition/specs/YYYY-MM-DD-<project>-phase<N>-<topic>.md`. Update roadmap status to `spec-generated`. Suggest: invoke `writing-plans` with this spec.

---

## Cross-Phase Changes

Auto-detect codebase drift. Confirm in mini-interview. If a change affects a completed phase, offer to include migration in current phase.

**Detection triggers:**
- New models/endpoints not in roadmap
- Modified API contracts from previous phases
- Changed dependencies or tech stack

**Response options:**
1. Include migration in current phase spec
2. Update roadmap and re-scope affected phases
3. Defer migration to a dedicated phase

Detailed scenarios: see [REFERENCE.md](REFERENCE.md).

## Validation Checklist

Before marking any output complete, verify:

**Roadmap validation:**
- [ ] Every phase is a vertical slice (DB + API + UI)
- [ ] 3-5 stories per phase (subdivide if >5, merge if <2)
- [ ] Dependencies form a DAG (no circular dependencies)
- [ ] No features invented beyond PRD scope
- [ ] User approved all priority/deferral decisions

**Spec validation:**
- [ ] All required sections present (components, data models, API contracts, user flows, acceptance criteria, dependencies)
- [ ] Under 300 lines (subdivide if exceeded)
- [ ] No implementation details (only contracts and interfaces)
- [ ] Acceptance criteria are independently verifiable
- [ ] Dependencies reference specific artifacts from previous phases

## Error Recovery

| Error | Recovery |
|-------|----------|
| PRD in unsupported format (PDF) | Request TXT/Markdown/HTML conversion. Do not attempt PDF parsing. |
| Codebase exploration fails (no package.json, access denied) | Skip codebase context. Note in roadmap: "Codebase structure unknown — verify assumptions in Phase 1." |
| Circular dependencies detected | Report to user. Propose phase reordering or dependency breaking. Do not auto-resolve. |
| PRD too short (1-2 features) | Suggest: "Use `brainstorming` + `writing-plans` directly. No decomposition needed." |
| Phase exceeds 300 lines | Subdivide phase. Update roadmap with new phase boundaries. |

## Critical Rules

- **ONE spec at a time**, just before execution. Never generate all phases at once.
- **Always interview.** Every PRD has assumptions. Never skip.
- **Every phase is a vertical slice.** No pure-backend or pure-UI phases.
- **DO NOT invent features** not in the PRD. DO NOT make priority calls without the user.

**No exceptions — including these rationalizations:**
- "User instructions take precedence" — does NOT override one-spec-at-a-time, strict reading rules, or the interview requirement.
- "Domain-inherent" or "implementation requirement" — is NOT a valid reason to add features absent from the approved roadmap. If it's not on the roadmap, ASK first.
- "User already answered informally" — does NOT replace the formal mini-interview. Ask the questions explicitly.
- "Given the deadline" — does NOT justify generating specs with unconfirmed assumptions. Wait for confirmation.
- "Phase 1 is safe to produce" — does NOT replace formal roadmap approval. No spec generation until the roadmap is explicitly approved, even if a phase seems unaffected by pending changes.
- "Hybrid approach" — is NOT a valid reason to batch interview questions. ONE question per message, always.
- "Time constraint trade-off" — is NOT a valid reason to batch questions or skip process steps. Speed at the cost of process integrity produces worse outcomes.

Full list of common mistakes and red flags: see [REFERENCE.md](REFERENCE.md).