# Project Slicing — Reference

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
delivered by Phase 3.

## Dependency Graph
```
Phase 1 (Auth + Onboarding)
  └→ Phase 2 (Shopify + Data Pipeline)
       ├→ Phase 3 (Anomaly Detection + Insight Delivery)
       │    └→ Phase 4 (Dashboard + Notifications)
       │         └→ Phase 5 (AI Chat)
       │              └→ Phase 6 (Settings + Team Management)
       └→ Phase 7 (Meta Ads Integration) ← parallelizable with Phases 3-6
```

## Cross-Cutting Concerns

- **Mobile responsive layout** — same priority as desktop (shoppable, not postage stamp)
- **Accessibility (WCAG 2.1 AA)** — keyboard navigation, screen reader labels, color contrast
- **Testing** — unit tests for anomaly detection and insight generation, integration tests for sync pipeline
- **Security** — Row Level Security (RLS) enabled on all tables from Phase 1
- **Monitoring** — Sentry (errors), LogSnag (logs), PostHog (product analytics)
- **Error handling** — toast notifications for UI, email alerts for critical sync failures

## Phase Table

| Phase | Name | Stories | Dependencies | Value | Done Criteria | Status |
|-------|------|---------|--------------|-------|---------------|--------|
| 1 | Auth + Onboarding | S1, S2, S3, S4, S5 | None | Base — no product without users | User can sign up, connect Shopify | pending |
| 2 | Shopify + Data Pipeline | S10, S17, S18, S19, S20 | Phase 1 | Core data flow — enables everything | Shopify data syncs every 6 hours | pending |
| 3 | Anomaly Detection + Insight Delivery | S24, S25, S28, S29, S30 | Phase 2 | Core differentiator — aha moment | User receives AI-produced insight email | pending |
| 4 | Dashboard + Notifications | S35, S36, S41, S43, S44 | Phase 3 | Visual confirmation — users see value in app | Morning Brief + insight history in app | pending |
| 5 | AI Chat | S58, S59, S60, S61, S62 | Phase 4 | Engagement — deeper interaction | Chat responds with data-grounded answers | pending |
| 6 | Settings + Team Management | S63, S64, S66, S67, S68 | Phase 5 | Retention — multi-user support | Team invite + RLS working | pending |
| 7 | Meta Ads Integration | S11, S12, S14, S15, S16 | Phase 2 | Depth — multi-source diagnosis | Meta data flows in with anomaly detection | pending |

## Deferred Features
- S6-S9: Onboarding progress, badges for connecting more sources (post-MVP polish)
- S13: Klaviyo integration (Phase 8, depends on Phase 2)
- S21-S23: Meta rate limiting, data normalization, sync logging (included in Phase 7 scope, not a separate story count)
- S26-S27: Anomaly detection scheduling and critical alerts (included in Phase 3 scope)
- S31-S34: Severity assignment, insight storage, action tracking (included in Phase 3 scope)
- S37-S40: Email and Slack notification tiers (included in Phase 4 scope)
- S45-S57: Performance view, Ask Avero buttons, channel breakdown (distributed across Phases 4-5)
- S65: Additional data sources (post-v1)
- S69-S73: Mobile responsiveness, accessibility (cross-cutting concerns)
- Google Ads, Stripe, GA4 integrations (Tier 2/3 — post-launch)
- Custom reports/dashboards (v2)
- Team collaboration features (comments, mentions)
- Mobile native app (responsive web in v1)

## Parallelizable Phases
- Phase 7 (Meta Ads Integration) can start after Phase 2 is complete. It does NOT depend on Phases 3-6. Even if executing sequentially (solo developer), Phases 3 and 7 have NO technical dependency on each other.
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

## Anti-Example — DO NOT Generate This

The following spec violates project-slicing rules. Every marked item belongs in `writing-plans`:

```markdown
# ❌ BAD Spec Example

## Components
- `(auth)/signup/page.tsx` — Formulario de registro ← FILE PATH, not component name
- `app/api/billing/checkout/route.ts` — Stripe checkout ← FILE PATH

## Data Models
**`tenants`**
| Campo | Tipo | ← SQL DDL with specific types
| `id` | `bigint GENERATED ALWAYS AS IDENTITY` |
| `plan_tier` | `text CHECK ('starter', 'pro', 'growth')` |

## Setup
npx create-next-app@latest myapp --typescript ← SETUP COMMAND
npm install @supabase/supabase-js stripe ← PACKAGE INSTALL

**Directory structure:**
src/
  app/
    (auth)/signup/page.tsx ← DIRECTORY TREE
```

**✅ CORRECT equivalent:**
```markdown
## Components
- `AuthService` — handles registration, login, OAuth
- `BillingService` — plan selection and Stripe checkout

## Data Models
- Tenant { id: bigint, name: string, planTier: enum(starter|pro|growth), subscriptionStatus: enum(trial|active|canceled) }

## Tech Stack
- Next.js 14 (App Router, TypeScript)
- Supabase (Auth, Database, RLS)
- Stripe (Subscriptions, Billing)
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
7. **Tech Stack** (optional) — list of technologies for this phase. Helps `writing-plans` know which domain skills to load. Write "N/A" if no specific tech stack.

Max 300 lines per spec. If it exceeds 300, the phase scope is too broad — subdivide and update the roadmap. **DO NOT trim content to fit under 300 lines.** If the spec naturally exceeds 300 lines, the phase must be split into two smaller phases, each with its own spec.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Generating all phase specs at once | ONE spec at a time, just before execution |
| Skipping the interview | Always interview — every PRD has assumptions |
| Horizontal phases (all DB, then all API) | Every phase must be a vertical slice |
| Phase whose output only the next phase consumes (invisible middleware) | Merge it with the consuming phase. If Phase N produces data that Phase N+1 displays, they belong together |
| Priority decisions without asking | Business value is the user's call |
| Reading entire codebase for context | Follow STRICT reading rules (see SKILL.md SPEC MODE step 1) |
| Inventing features ("best practice says...") | DO NOT add features not in PRD |
| Phase with 10+ stories | Subdivide. Max 5 per phase. |
| Phase with 1-2 stories that could merge | Merge with adjacent phase unless: (1) domain is coherent (passes one-sentence test), (2) it's a vertical slice, and (3) merging would break domain coherence. |
| Counting themes instead of stories | "5 themes with 6 stories each" = 30 stories. Max 5 stories per phase, counted individually. |
| Using ranges to hide story count | "Stories 1-9" is 9 stories. List individually: S1, S2, S3, S4, S5 = 5 stories = 1 phase. S6-S9 go in next phase. |
| Calling "setup" or "infrastructure" a story | Setup is a cross-cutting concern, not a story. Track it in "Cross-Cutting Concerns" section, not in story count. |
| Grouping unrelated domains in one phase | Each phase should have ONE core domain. "AI Chat + Billing + Teams" fails the one-sentence test. Split into separate phases. |
| Creating a "catch-all" final phase with leftovers | Polish, testing, and launch are cross-cutting concerns. Billing, teams, and AI Chat are distinct domains. Don't combine them. |
| Phase name with multiple "+" or "and" connecting unrelated domains | "AI Chat + Billing & Teams" signals 3 domains that should be separate phases. Use the one-sentence test. |
| Batching interview questions in one message | ONE question per message. Wait for answer. Then next. |
| Asking interview questions without using the `question` tool | ALWAYS use the `question` tool for interviews. It provides structured options and clearer responses. |
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
| Including file paths and directory structure | Specs use component names, not file paths. `(app)/home/page.tsx` → `HomePage — placeholder for dashboard content` |
| Including SQL DDL with types and constraints | Specs use abstract types: `Tenant { id: bigint, planTier: enum(...) }` not `CREATE TABLE` with `CHECK` constraints |
| Including setup commands and env vars | These belong in writing-plans where the tech stack is defined and domain skills are loaded |
| Adding "Tech Stack" section with implementation details | Tech Stack lists technologies only (Next.js, Supabase, Stripe), not how to configure them |
| Including schema or table mappings in roadmap | Roadmap lists tech stack names only. writing-plans reads PRD directly for schema details |
| Trimming spec content to fit under 300 lines | If spec exceeds 300 lines, subdivide the phase. Do not remove content to meet the limit |
| Counting themes instead of stories | "5 themes with 6 stories each" = 30 stories. Max 5 stories per phase, counted individually. |
| Using ranges to hide story count | "Stories 1-9" is 9 stories. List individually: S1, S2, S3, S4, S5 = 5 stories = 1 phase. S6-S9 go in next phase. |
| Calling "setup" or "infrastructure" a story | Setup is a cross-cutting concern, not a story. Track it in "Cross-Cutting Concerns" section, not in story count. |
| Grouping unrelated domains in one phase | Each phase should have ONE core domain. "AI Chat + Billing + Teams" fails the one-sentence test. Split into separate phases. |
| Creating a "catch-all" final phase with leftovers | Polish, testing, and launch are cross-cutting concerns. Billing, teams, and AI Chat are distinct domains. Don't combine them. |
| Phase name with multiple "+" or "and" connecting unrelated domains | "AI Chat + Billing & Teams" signals 3 domains that should be separate phases. Use the one-sentence test. |
| Including data models for features in later phases | Each data model must trace to a PRD story in THIS phase. ChatSession in Phase 1 when Chat is Phase 4 = feature leakage. Remove it. |
| Including nullable "future" columns (e.g., ad_spend in Shopify-only phase) | Add columns when the phase that needs them is executed, not "for when we add X later." |
| Including API endpoints for future-phase features | Notification settings endpoint in Phase 1 when notifications are Phase 3 = remove from Phase 1 spec. |
| Listing testing or mobile responsiveness as a story | These are cross-cutting concerns listed in the Cross-Cutting Concerns section, not individual stories. |
| Defaulting to linear phase sequence when phases are parallelizable | If Phase B and C both depend only on Phase A, they are parallelizable. Mark them. Sequential execution is an implementation detail. |
| Stretching Phase 1 to deliver the full "aha moment" | Phase 1 delivers minimum viable user value. Aha moment spanning Phase 1 + 2 is correct. Max 5 stories per phase. |
| Phase containing more than one external service integration | At most ONE integration per phase. Shopify in Phase 1, Meta in Phase 2, etc. |

## Red Flags — STOP Immediately If You Catch Yourself

- Generating specs for more than one phase at a time
- Skipping the interview because "the PRD is clear enough"
- Asking interview questions without using the `question` tool
- Creating a phase with no UI component (not a vertical slice)
- Adding a feature not mentioned in the PRD or roadmap
- Reading test files or full component source code for context
- Done criteria that describe internal system state ("system produces X") rather than user-observable behavior ("user sees/receives X")
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
- Including file paths or directory structures in specs ("this helps writing-plans")
- Including SQL DDL with types, constraints, or indexes in specs ("implementation context")
- Including setup commands, env vars, or code in specs ("being thorough")
- Including schema or table mappings in roadmap ("helps writing-plans know what to create")
- Trimming spec content to fit under 300 lines instead of subdividing the phase
- A phase name with multiple "+" or "and" connecting unrelated domains (e.g., "AI Chat + Billing & Teams")
- A final phase named "Polish", "Testing", "Launch", or "Cleanup" — these are cross-cutting concerns
- Story ranges like "1-9" that hide >5 stories in one phase
- Data models or API endpoints in a spec that serve future-phase stories
- Nullable columns "for when we add X later" in data models
- Phase 1 containing more than one external service integration
- Phase 1 containing LLM/AI + full dashboard + notification delivery (too big)
- Phase 1 with >5 individual PRD stories
- All phases marked sequential when some share the same parent dependency
- Testing, mobile responsive, or accessibility listed as stories instead of cross-cutting concerns

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
| Phase spec exceeds 300 lines | Line count after generation | Subdivide phase into two. Update roadmap with new boundaries and dependencies. Re-generate spec for each sub-phase. **DO NOT trim content to fit under 300.** |
| Ambiguous PRD section | Multiple valid interpretations | Ask in interview. Never assume. Present options: "The PRD mentions X. Does this mean A or B?" |
| Codebase contradicts roadmap | Models/endpoints differ from roadmap assumptions | Report discrepancy. Ask: "Roadmap assumes X, but codebase has Y. Which is correct?" |

## Validation Checklist

Run through this checklist before presenting any output to the user.

### Roadmap Validation

| Check | How to verify |
|-------|---------------|
| Every phase is a vertical slice | Each phase includes DB changes + API endpoints + UI screens. Flag any phase with only one layer. |
| Every phase has user-visible output | For each phase, ask: "Can an end user see or interact with this phase's output without needing Phase N+1?" If not, merge with the consuming phase. Storing computed results a user can't yet see is NOT user-visible. |
| 3-5 stories per phase | Count user stories. If >5, subdivide. If <2, consider merging with adjacent phase. |
| Dependencies form a DAG | Walk dependency graph. If any cycle exists (A→B→A), report and propose reordering. |
| No invented features | Cross-reference every phase feature against the PRD. If a feature has no PRD source, remove it. |
| User-approved priorities | Verify every priority/deferral decision was confirmed by the user in the interview. |
| Functional deliverable per phase | Each phase must produce something the user can see/use. No "backend-only" or "infrastructure-only" phases. |
| Story count is individual PRD numbers | Open the PRD. Count each numbered story in each phase. If any phase has >5, subdivide. Ranges like "1-9" = 9 stories, not 1. Themes like "Auth" = count the stories inside. |
| Stories trace to PRD numbers | Every story in every phase maps to a specific PRD story number. No invented stories. No themes without PRD backing. |
| One domain per phase | Describe each phase in one sentence. If you need "and" or "+" more than once, split the phase. |
| No cross-cutting phase | Testing, mobile, accessibility, security, monitoring, infrastructure setup are in "Cross-Cutting Concerns" section, NOT stories in any phase. |
| Phase 1 scope | Max 5 stories. Max 1 external service integration. No LLM/AI unless the product IS AI-first (and even then, max 5 stories). |
| Parallelization analyzed | Phases sharing the same parent dependency but not depending on each other are marked as parallelizable. |
| No feature leakage in specs | For each data model, API endpoint, and acceptance criterion, trace it to a specific PRD story number in this phase. Remove anything that traces to a future phase. |
| No nullable future columns in data models | If a column exists "for when we add X later," it belongs in the X phase spec, not this one. |

### Spec Validation

| Check | How to verify |
|-------|---------------|
| All 6 required sections present | Components, Data Models, API Contracts, User Flows, Acceptance Criteria, Dependencies |
| Under 300 lines | `wc -l` on spec file. If exceeded, subdivide phase. |
| No implementation details | Scan for code blocks with logic (queries, conditionals, loops). Only contracts and interfaces allowed. |
| Acceptance criteria verifiable | Each criterion can be tested independently. "System works well" is not valid. "User can login with email/password and receive JWT" is valid. |
| Dependencies reference specifics | Must name exact models, endpoints, or artifacts from previous phases. "Depends on Phase 1" is not specific enough. |
| No features beyond roadmap scope | Cross-reference spec features against roadmap phase entry. Flag anything not listed. |
| No file paths | Scan for `/`, `.tsx`, `.ts`, `.sql`, `src/`, `app/` patterns. Zero file paths allowed. |
| No SQL DDL | Scan for `CREATE TABLE`, `ALTER`, `GENERATED`, `CHECK`, `CONSTRAINT`. Zero DDL allowed. Abstract types like `bigint` in model definitions are OK. |
| No code blocks with logic | Only JSON request/response shapes allowed in code blocks. No functions, queries, or conditionals. |
| No setup commands | Scan for `npm`, `npx`, `pip`, `cargo`, `brew`, etc. Zero setup commands allowed. |
| No env variables | Scan for `NEXT_PUBLIC_`, `SECRET_`, `_KEY`, `_URL` patterns. Zero env vars allowed. |
| No directory trees | Scan for tree-like structures (`src/`, `app/`, indented file lists). Zero directory trees allowed. |
| No features beyond roadmap scope | Cross-reference spec features against roadmap phase entry. Flag anything not listed. |
| No feature leakage beyond phase scope | For each data model, API endpoint, and acceptance criterion, trace it to a PRD story number in THIS phase. Remove anything that traces to a future phase. |
| No nullable future columns | If a column exists "for when we add Meta later," it belongs in the Meta phase spec, not this one. |