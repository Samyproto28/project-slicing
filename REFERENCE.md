# Project Decomposition — Reference

## Contents
- Edge cases
- Phase status lifecycle
- Roadmap example
- Phase spec example
- Spec format specification
- Common mistakes
- Red flags
- Cross-phase changes
- Error recovery
- Validation checklist

Detailed examples, edge cases, and format specifications. See [SKILL.md](SKILL.md) for the core workflow and rules.

## Edge Cases

| Case | How to handle |
|------|---------------|
| PRD too short (1-2 features) | Suggest: "This is small enough for `brainstorming` + `writing-plans` directly. No decomposition needed." |
| PRD without clear user stories | In the interview, ask: "What should the user be able to do on this screen?" to extract user stories before decomposing. |
| Feature depending on external service (Stripe, OAuth) | Create a dedicated integration phase with the service as an explicit dependency. Do not bury it inside another phase. |
| User wants to skip a phase | Allow out-of-order execution but warn: "Phase N depends on Phase M. If you skip it, the spec will assume Phase M artifacts already exist." |
| Two phases are independent | Mark them as parallelizable in the roadmap. User can execute in any order or in parallel. |

## Phase Status Lifecycle

Each phase transitions through four statuses:

```
pending → spec-generated → in-progress → completed
```

- **pending**: Phase exists in the roadmap but no spec has been generated yet.
- **spec-generated**: Spec has been created and approved by the user. Ready for `writing-plans`.
- **in-progress**: Phase is being executed with `writing-plans`.
- **completed**: Phase is done. All acceptance criteria met.

**Determining the next phase:** When the user says "next phase" without specifying which one, find the first phase whose status is `pending` and ALL of its dependency phases have status `completed`.

## Roadmap Example

```markdown
# Avero MVP - Development Roadmap

## Executive Summary
Avero is an AI-first analytics platform for e-commerce merchants ($500K-$5M ARR).
7 phases, estimated 3-5 months solo. Core value (anomaly detection + insights)
delivered by Phase 4.

## Dependency Graph
```
Phase 1 (Auth + Onboarding)
  └→ Phase 2 (Shopify + Data Pipeline)
       ├→ Phase 3 (Anomaly Detection + Basic Insights)
       │    └→ Phase 4 (Dashboard + Notifications)
       │         └→ Phase 5 (AI Chat + Insights Detail)
       │              └→ Phase 6 (Settings + Team Management)
       └→ Phase 7 (Meta + Klaviyo Integrations)
```

## Phase Table

| Phase | Name | Stories | Dependencies | Value | Done Criteria | Status |
|-------|------|---------|--------------|-------|---------------|--------|
| 1 | Auth + Onboarding | 1-5 | None | Base — no product without users | User can sign up, connect Shopify | pending |
| 2 | Shopify + Data Pipeline | 10-12, 17-23 | Phase 1 | Core data flow — enables everything | Shopify data syncs every 6 hours | pending |
| 3 | Anomaly Detection + Insights | 24-34 | Phase 2 | Core differentiator | System produces structured insights | pending |
| 4 | Dashboard + Notifications | 35-42, 43-49 | Phase 3 | Aha moment — users see value | Morning Brief + email delivery | pending |
| 5 | AI Chat + Insights Detail | 50-54, 58-62 | Phase 4 | Engagement — deeper interaction | Chat responds with data-grounded answers | pending |
| 6 | Settings + Team Management | 63-68 | Phase 5 | Retention — multi-user support | Team invite + RLS working | pending |
| 7 | Meta + Klaviyo Integrations | 13-16 | Phase 2 | Depth — multi-source diagnosis | Meta + Klaviyo data flows in | pending |

## Deferred Features
- Google Ads, Stripe, GA4 integrations (Tier 2/3 — post-launch)
- Custom reports/dashboards (v2)
- Team collaboration features (comments, mentions)
- Mobile native app (responsive web in v1)

## Parallelizable Phases
- Phase 7 can start after Phase 2 (does not depend on Phases 3-6)
```

## Phase Spec Example

```markdown
# Avero - Phase 1: Auth & Onboarding

## Components
- `AuthService` — handles registration, login, JWT token management
- `UserService` — CRUD operations for user accounts
- `OnboardingService` — orchestrates first-time setup flow
- `PlanService` — plan selection and trial management

## Data Models
- User { id: bigint, email: string, passwordHash: string, role: enum(owner|admin|viewer), createdAt: timestamp }
- Tenant { id: bigint, name: string, plan: enum(starter|pro|growth), createdAt: timestamp }
- Membership { id: bigint, userId: bigint, tenantId: bigint, role: enum, joinedAt: timestamp }

## API Contracts
- POST /api/auth/register { email, password } → { token, user }
- POST /api/auth/login { email, password } → { token, user }
- GET /api/auth/me → { user, tenant }
- POST /api/tenants/{id}/plan { plan } → { tenant }

## User Flows
1. User visits /register → enters email + password → receives verification email
2. User clicks verification link → account created, tenant created with starter plan
3. 7-day free trial starts (no credit card required)
4. User prompted to connect Shopify (first data source)

## Acceptance Criteria
- [ ] User can register with email/password
- [ ] User can login and receive JWT token
- [ ] Tenant is created with default plan on registration
- [ ] 7-day trial starts without credit card
- [ ] Onboarding prompts Shopify connection after registration

## Dependencies on Previous Phases
None (base phase)
```

## Spec Format Specification

Phase spec filenames: `YYYY-MM-DD-<project>-phase<N>-<topic>.md`

Required sections in order:
1. **Components** — name + one-line responsibility
2. **Data Models** — entities, fields, relationships. NO queries, NO business logic. Write "N/A" if phase doesn't touch DB.
3. **API Contracts** — method, path, request body, response body. NO implementation. Write "N/A" if phase doesn't add endpoints.
4. **User Flows** — numbered steps the user follows
5. **Acceptance Criteria** — checkbox items, each independently verifiable
6. **Dependencies on Previous Phases** — specific artifacts used (model names, endpoint paths). Write "None (base phase)" for Phase 1.

Max 300 lines per spec. If it exceeds 300, the phase scope is too broad — subdivide and update the roadmap.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Generating all phase specs at once | ONE spec at a time, just before execution |
| Skipping the interview | Always interview — every PRD has assumptions |
| Horizontal phases (all DB, then all API) | Every phase must be a vertical slice |
| Priority decisions without asking | Business value is the user's call |
| Reading entire codebase for context | Follow STRICT reading rules (see SKILL.md SPEC MODE step 1) |
| Inventing features ("best practice says...") | DO NOT add features not in PRD |
| Phase with 10+ stories | Subdivide. Max 5 per phase. |
| Batching interview questions in one message | ONE question per message. Wait for answer. Then next. |
| Making priority or deferral decisions alone | Only defer or prioritize features the user explicitly confirms |
| Batching SPEC MODE mini-interview questions | The 2 base questions can go together. Any additional questions (scope changes, drift, ambiguities) must be ONE per message, same as ROADMAP MODE |
| Making unilateral technical defaults under time pressure | "Will default to X" is a priority decision. Ask the user, even if they said "the PRD is clear" |
| Using "user instructions take precedence" to override process rules | User instructions do NOT override one-spec-at-a-time, strict reading, or interview requirements |
| Classifying features as "domain-inherent" to add them without approval | If it's not on the approved roadmap, it's a new feature. ASK first, regardless of how "obvious" it seems |
| Treating "follow-ups" to base mini-interview questions as part of the base | A follow-up is still an additional question. ONE per message. |
| Accepting user's codebase summary instead of reading yourself | The user's summary may miss critical details. Read the allowed files yourself. |
| Generating specs with unconfirmed assumptions due to deadline pressure | Generate with confirmed scope only. Flag unconfirmed items as "pending user confirmation." |
| Generating Phase 1 spec because it's "safe to produce" without roadmap approval | No spec generation until the roadmap is explicitly approved. Even stable phases must wait. |
| Using "hybrid approach" to justify batching interview questions | ONE question per message. "Hybrid" is just batching with a nicer name. |
| Using "time constraint trade-off" to justify batching or skipping steps | Speed at the cost of process integrity produces worse outcomes. ONE question per message, always. |

## Red Flags — STOP Immediately If You Catch Yourself

- Generating specs for more than one phase at a time
- Skipping the interview because "the PRD is clear enough"
- Creating a phase with no UI component (not a vertical slice)
- Adding a feature not mentioned in the PRD or roadmap
- Reading test files or full component source code for context
- Making a priority call without asking the user
- Asking multiple interview questions in one message
- Proceeding to SPEC MODE before the user approves the roadmap
- Batching additional SPEC MODE questions beyond the 2 base mini-interview questions
- Making unilateral technical defaults ("will default to polling/SSE/route group") without asking
- Saying "user instructions take precedence" to justify skipping process rules
- Classifying features as "domain-inherent" or "implementation requirement" to bypass the no-inventing rule
- Treating a "follow-up" to a mini-interview base question as not needing its own message
- Accepting the user's codebase summary instead of doing your own strict-scope read
- Generating specs with unconfirmed assumptions because of deadline pressure
- Generating a spec because the phase is "safe to produce" without formal roadmap approval
- Calling batched questions a "hybrid approach" to avoid the one-at-a-time rule
- Justifying batching as a "time constraint trade-off" — it's still batching

**All of these mean: Go back. Follow the process.**

## Cross-Phase Changes

When generating a spec for Phase N, the codebase may have diverged from what the roadmap assumed.

**Detection signals:**
- New models, endpoints, or services not listed in the roadmap
- Modified API contracts from previous phases (different request/response shapes)
- Changed tech stack or dependencies (new framework, different ORM)
- Removed features that a later phase depends on

**Handling procedure:**
1. **Detect**: During SPEC MODE context reading, compare codebase against roadmap assumptions
2. **Report**: Tell the user exactly what changed and which phases are affected
3. **Propose**: Offer one of three options:
   - Include migration/compatibility work in current phase spec
   - Update roadmap and re-scope affected future phases
   - Create a dedicated migration phase
4. **Confirm**: Never auto-apply changes. User decides the approach.

**Example**: If Phase 2 changed the User model to add `organizationId`, and Phase 4's spec assumed the old schema, report: "Phase 2 added `organizationId` to User. Phase 4 spec needs to account for this. Include migration in Phase 4 or update roadmap?"

## Error Recovery

| Error | Detection | Recovery |
|-------|-----------|----------|
| PRD in PDF format | File extension is .pdf | Request TXT/Markdown/HTML. Do not attempt parsing. Say: "I can only process TXT, Markdown, or HTML. Please convert and retry." |
| Codebase inaccessible | `ls` or `glob` returns empty/error | Skip codebase context. Add to roadmap: "Codebase structure unknown — verify technical assumptions in Phase 1." |
| No package.json | Glob returns no results | Note tech stack as "unknown". Ask in interview: "What tech stack are you using?" |
| Circular dependencies | Phase A depends on B, B depends on A | Report to user with graph. Propose: merge phases, extract shared dependency into earlier phase, or break dependency. Do not auto-resolve. |
| PRD too short (1-2 features) | Count distinct features/user stories | Suggest: "This is small enough for `brainstorming` + `writing-plans` directly. No decomposition needed." |
| Phase spec exceeds 300 lines | Line count after generation | Subdivide phase into two. Update roadmap with new boundaries and dependencies. Re-generate spec for each sub-phase. |
| Ambiguous PRD section | Multiple valid interpretations | Ask in interview. Never assume. Present options: "The PRD mentions X. Does this mean A or B?" |
| Codebase contradicts roadmap | Models/endpoints differ from roadmap assumptions | Report discrepancy. Ask: "Roadmap assumes X, but codebase has Y. Which is correct?" |

## Validation Checklist

Run through this checklist before presenting any output to the user.

### Roadmap Validation

| Check | How to verify |
|-------|---------------|
| Every phase is a vertical slice | Each phase includes DB changes + API endpoints + UI screens. Flag any phase with only one layer. |
| 3-5 stories per phase | Count user stories. If >5, subdivide. If <2, consider merging with adjacent phase. |
| Dependencies form a DAG | Walk dependency graph. If any cycle exists (A→B→A), report and propose reordering. |
| No invented features | Cross-reference every phase feature against the PRD. If a feature has no PRD source, remove it. |
| User-approved priorities | Verify every priority/deferral decision was confirmed by the user in the interview. |
| Functional deliverable per phase | Each phase must produce something the user can see/use. No "backend-only" or "infrastructure-only" phases. |

### Spec Validation

| Check | How to verify |
|-------|---------------|
| All 6 required sections present | Components, Data Models, API Contracts, User Flows, Acceptance Criteria, Dependencies |
| Under 300 lines | `wc -l` on spec file. If exceeded, subdivide phase. |
| No implementation details | Scan for code blocks with logic (queries, conditionals, loops). Only contracts and interfaces allowed. |
| Acceptance criteria verifiable | Each criterion can be tested independently. "System works well" is not valid. "User can login with email/password and receive JWT" is valid. |
| Dependencies reference specifics | Must name exact models, endpoints, or artifacts from previous phases. "Depends on Phase 1" is not specific enough. |
| No features beyond roadmap scope | Cross-reference spec features against roadmap phase entry. Flag anything not listed. |