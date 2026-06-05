# Avero v1 — Phased Roadmap

## Executive Summary

Avero is an AI-first analytics platform for DTC e-commerce merchants ($500K–$5M ARR) that proactively diagnoses why metrics change across Shopify, Meta Ads, and Klaviyo, delivering actionable recommendations instead of requiring merchants to interpret dashboards. This roadmap decomposes v1 into 5 phases, each delivering functional value as a vertical slice.

**Core MVP thesis:** The differentiator is _proactive insight delivery_ — not dashboards, not chat, not team collaboration. The fastest path to that value is: sign up → connect Shopify → data flows in → anomalies detected → first Morning Brief email. Everything else deepens or broadens that core loop.

**Prioritization calls:**
1. AI Chat (Stories 58-62) deferred — proactive insight email is MVP value; chat deepens engagement post-validation
2. Team management with roles/invites (Stories 67-68) deferred — single-user MVP; RLS infrastructure built from Phase 1 for data isolation, but no invites/roles UI until post-launch
3. Shopify-only for Phases 1-4 — Meta + Klaviyo integration is a parallel workstream after the data pipeline is stable (Phase 6)
4. Email delivery joins anomaly detection — the "aha moment" requires _receiving_ an insight, not just generating one
5. Auth + Shopify Connection grouped as Phase 1 — separately, Auth is a thin vertical slice with no business value; together, users complete onboarding and see data flowing

## Founder Validation Flags

These assumptions should be confirmed before Phase 1 begins. Each has revenue implications:

1. **AI Chat in Pro/Growth tiers** — Your pricing model lists AI Chat as a Pro/Growth differentiator. Deferring it means Pro launches without a listed differentiator. Is the daily insight email sufficient differentiator for Pro at launch?
2. **Shopify-only MVP** — Your PRD says Meta + Klaviyo are Tier 1, but cross-source diagnosis isn't possible until Phase 6. Phases 2-4 insights will be Shopify-only (revenue, orders, AOV — no ROAS, no email metrics). Are Shopify-only insights sufficient for early validation?
3. **Single-user launch** — Deferring team invites/roles means early adopters get a private account. Your PRD already scopes out team collaboration features, but are there immediate prospects who need multi-user access?
4. **Performance Dashboard timing** — Stories 55-57 (Performance view with charts, date picker, ad campaigns table) are deferred. The core MVP (Phases 1-4) delivers insights via email only; Phase 5 adds in-app dashboard. Are you comfortable with no in-app charts until Phase 5?

## Dependency Graph

```
Phase 1 (Auth + Data Connection)
  └── Phase 2 (Data Pipeline)
        └── Phase 3 (Core Value — Detection + Insights)
              └── Phase 4 (Insight Delivery + Email)
                    └── Phase 5 (Morning Brief Dashboard)

Phase 6 (Meta + Klaviyo Integrations) — starts after Phase 2, parallel to Phases 3-5
```

Phase 2 is the critical path — all data-dependent phases branch from it. Phase 6 (Meta + Klaviyo) can begin once the data pipeline foundation exists but does not block the core insight loop.

## Phase Table

| Phase | Topic | Stories | Sub-phases | Dependencies | Value Delivered | Status |
|-------|-------|---------|------------|--------------|----------------|--------|
| 1 | Auth + Data Connection | 1-5, 10-12 | 1a: Auth (1-5), 1b: Shopify Connection (10-12) | None | Users can sign up, connect Shopify, see data flowing | pending |
| 2 | Data Pipeline | 17-23 | 2a: Sync + Ingestion (17-20), 2b: Processing + Logging (21-23) | Phase 1 | Data flows — enables anomaly detection | pending |
| 3 | Core Value — Detection + Insights | 24-31 | 3a: Anomaly Detection (24-27), 3b: Insight Generation (28-31) | Phase 2 | Engine detects anomalies and generates structured insights | pending |
| 4 | Insight Delivery + Email | 32-38 | 4a: Insight Detail + Tracking (32-34), 4b: Email Notifications (35-38) | Phase 3 | **Aha moment** — first Morning Brief email | pending |
| 5 | Morning Brief Dashboard | 43-49 | 5a: Dashboard Core (43-46), 5b: Dashboard Detail (47-49) | Phase 4 | In-app experience — Morning Brief with metrics and charts | pending |
| 6 | Meta + Klaviyo Integrations | 11-16 | 6a: Meta Ads (11-12, 14), 6b: Klaviyo (13, 15-16) | Phase 2 | Multi-source diagnosis — deeper root causes | pending |

## Phase Summaries

### Phase 1: Auth + Data Connection (Stories 1-5, 10-12)

**Sub-phase 1a: Auth + Onboarding (Stories 1-5) — 5 stories**
Sign up (email/Google OAuth via Supabase Auth), 7-day free trial without credit card, plan selection (Starter/Pro/Growth), prompt to connect Shopify as first data source, sync progress indicator. Establishes multi-tenant model with RLS policies from day one (tenant isolation enforced, but no team invites UI until post-launch).

**Vertical slice:** DB (users, tenants, memberships tables) + API (register, login, /me endpoints) + UI (signup, login, plan selection, onboarding flow pages).

**Done when:** User can register, select a plan, and see the Shopify connection prompt.

**Sub-phase 1b: Shopify Connection (Stories 10-12) — 3 stories**
Shopify OAuth connection with scoped permissions (`read_orders, read_products, read_customers, read_analytics`), Meta Ad Account + Pixel selection UI (placeholder for Phase 6 — enables future multi-source setup), connection status display.

**Vertical slice:** DB (connections table with encrypted tokens via Supabase Vault) + API (OAuth callback, connection CRUD endpoints) + UI (Connect Shopify button, OAuth redirect, connection success page).

**Done when:** User can connect their Shopify store and see a "connected" status.

**Combined Phase 1 deliverable:** Merchants can create an account, connect Shopify, and see data sync starting. This is a meaningful vertical slice — users land in the product and see it working, not an empty dashboard.

### Phase 2: Data Pipeline (Stories 17-23)

**Sub-phase 2a: Sync + Ingestion (Stories 17-20) — 4 stories**
Incremental 6-hour syncs via Inngest cron (every active connection), full 90-day sync on initial connection, raw API payload storage in `raw_data` jsonb column, sync job logging in `sync_jobs` + `sync_logs` tables.

**Vertical slice:** DB (shopify_orders, shopify_products, shopify_customers tables with raw_data jsonb, sync_jobs table) + API (sync trigger endpoints, sync status endpoint) + UI (sync progress indicator from Phase 1 now shows real data, sync health status).

**Done when:** Shopify data syncs on schedule and raw API data is queryable.

**Sub-phase 2b: Processing + Logging (Stories 21-23) — 3 stories**
UTC/currency normalization for cross-source accuracy, sync job logging (records_synced, errors, duration) with `sync_logs`, Meta API rate limit throttling and exponential backoff (infrastructure ready for Phase 6 Meta integration).

**Vertical slice:** DB (daily_metrics materialized views, normalized data tables) + API (metrics query endpoints with UTC-normalized responses) + UI (data freshness indicators, sync error display).

**Done when:** Data is normalized, queryable, and sync errors are visible.

**Combined Phase 2 deliverable:** Shopify data syncs every 6 hours, is normalized and queryable. The pipeline is production-ready for Phase 6 (Meta + Klaviyo).

### Phase 3: Core Value — Detection + Insights (Stories 24-31)

**Sub-phase 3a: Anomaly Detection (Stories 24-27) — 4 stories**
Daily KPI baseline calculation (30-day rolling avg + stddev) for revenue, orders, AOV, ROAS, MER, email metrics. Anomaly flagging (2σ + 20% WoW). Daily detection schedule at 8:00 AM (Pro/Growth, weekly for Starter). Critical anomaly detection runs on every sync cycle independent of schedule (revenue drops >30% trigger immediate alerts regardless of tier).

**Vertical slice:** DB (daily_metrics baselines, anomaly flags on metric records) + API (detection trigger endpoint, anomaly list endpoint, baseline read endpoint) + UI (internal: Inngest dashboard shows detection jobs running; external: anomaly data available for Phase 4 email + Phase 5 dashboard).

**Done when:** Anomalies are detected daily and stored. Detection job runs on schedule and produces anomaly records.

**Sub-phase 3b: Insight Generation (Stories 28-31) — 4 stories**
Consolidate all detected anomalies into a single report per period (daily Pro/Growth, weekly Starter). Send each report to LLM with structured context (last 7 days metrics, anomalies, connected sources). LLM returns structured JSON (`title`, `summary`, `recommendations`, `severity`, `type`). Severity assignment: Critical > High > Medium > Low > Normal (cascading rules from PRD).

**Vertical slice:** DB (insights table with full context + recommendations as jsonb) + API (insight generation trigger, insight list/detail endpoints) + UI (internal: Inngest dashboard shows generation jobs; external: stored insights consumable by Phase 4 email and Phase 5 dashboard).

**Done when:** System produces structured insights with severity, title, summary, and recommendations on schedule.

**Combined Phase 3 deliverable:** The intelligence engine is running. Anomalies are detected, insights are generated with LLM root-cause analysis, and structured data is stored. This is the core differentiator in production — even without email delivery or dashboard, the system is generating value from data.

### Phase 4: Insight Delivery + Email (Stories 32-38)

**Sub-phase 4a: Insight Detail + Tracking (Stories 32-34) — 3 stories**
Severity rules implementation (Critical/High/Medium/Low/Normal cascading thresholds from PRD). Insight storage with full context (anomalies detected, metrics used, recommendations) in `insights` table. Merchant action tracking (`insight_actions`: viewed, dismissed, marked_as_done).

**Vertical slice:** DB (insights table with context jsonb, insight_actions table) + API (insight detail endpoint, action tracking endpoints: POST /api/insights/{id}/action) + UI (an insight can be viewed, dismissed, or marked done — even as a simple API response, the tracking mechanism exists for Phase 5 dashboard).

**Done when:** Insights are fully stored with context and severity, and action tracking works via API.

**Sub-phase 4b: Email Notifications (Stories 35-38) — 4 stories**
Weekly email (Starter, Mondays at 8 AM), daily email (Pro/Growth, 8 AM), immediate email alert for Critical-severity insights (all tiers). Email format: severity badge, title, root causes, 2-4 actionable recommendations. "All Clear" briefing when no anomalies detected. Templates via Resend with tier-aware scheduling (Inngest cron).

**Vertical slice:** DB (email_logs table tracking delivery status) + API (email trigger endpoints, email preferences read/write) + UI (formatted Morning Brief email delivered to merchant's inbox — this IS the user-facing UI).

**Done when:** Merchant receives their first Morning Brief email with detected anomalies, root causes, and actionable recommendations. **This is the aha moment.**

**Combined Phase 4 deliverable:** Merchants receive proactive insight emails. The core value proposition — "nobody proactively tells the merchant what happened and what to do" — is now delivered. Even without the in-app dashboard, merchants are getting value from Avero.

### Phase 5: Morning Brief Dashboard (Stories 43-49)

**Sub-phase 5a: Dashboard Core (Stories 43-46) — 4 stories**
Home view displays current Morning Brief (today's consolidated insight report) as first thing the user sees. Severity badge (🔴🟠🟡🟢✅), title, root causes, numbered recommendations. "Ask Avero about this" button present but non-functional (pointing to deferred AI Chat feature). Four Quick Metrics cards (Revenue, Orders, AOV, MER) with delta percentages.

**Vertical slice:** DB (no new tables — reads from insights + daily_metrics) + API (GET /api/morning-brief, GET /api/quick-metrics) + UI (Home page with Morning Brief card + Quick Metrics grid).

**Done when:** Merchant opens the app and immediately sees today's Morning Brief with headline metrics.

**Sub-phase 5b: Dashboard Detail (Stories 47-49) — 3 stories**
7-day revenue trend chart (Tremor AreaChart). Channel Breakdown section with cards for Shopify, Meta Ads, and Klaviyo (Meta/Klaviyo cards show "Connect in Settings" CTA until Phase 6). Recent Insights list (last 5) with severity badges and timestamps.

**Vertical slice:** DB (no new tables — reads from daily_metrics + channel_metrics) + API (GET /api/revenue-trend, GET /api/channel-breakdown, GET /api/recent-insights) + UI (chart section, channel cards, recent insights list).

**Done when:** Merchant can see revenue trends, channel breakdown, and recent insight history in-app.

**Combined Phase 5 deliverable:** In-app Morning Brief experience. Merchants no longer rely solely on email — they can open Avero and immediately see what needs attention, why, and what to do. This is the full "aha moment" with UI.

### Phase 6: Meta + Klaviyo Integrations (Stories 11-16)

**Sub-phase 6a: Meta Ads (Stories 11-12, 14) — 3 stories**
Meta Ads OAuth connection via Marketing API, multi-account + pixel selection for merchants with multiple ad accounts, connection status display in Settings (last synced, active/error). Data pipeline extensions for `meta_ad_accounts`, `meta_campaigns`, `meta_insights`.

**Vertical slice:** DB (meta_ad_accounts, meta_campaigns, meta_insights tables) + API (Meta OAuth endpoints, connection CRUD) + UI (Connect Meta Ads button, account/pixel selector, status display in Settings).

**Done when:** Merchant can connect Meta Ads, select account/pixel, and data syncs every 6 hours.

**Sub-phase 6b: Klaviyo + Health Dashboard (Stories 13, 15-16) — 3 stories**
Klaviyo OAuth connection, sync error email notifications, one-click reconnect for broken connections. Data pipeline extensions for `klaviyo_profiles`, `klaviyo_campaigns`, `klaviyo_events`. Connection health dashboard showing all connections' status.

**Vertical slice:** DB (klaviyo_* tables) + API (Klaviyo OAuth, connection health endpoints) + UI (Connect Klaviyo button, error notification preferences, health dashboard with reconnect buttons).

**Done when:** Merchant can connect Klaviyo, see all connection statuses, and reconnect broken connections with one click.

**Combined Phase 6 deliverable:** Multi-source diagnosis. Insights now reference Meta ROAS and Klaviyo deliverability alongside Shopify revenue. Root-cause analysis becomes cross-source: "Revenue dropped 23% because Meta ROAS fell 40% on campaign X."

## Deferred Features

| Feature | Stories | Reason | Revisit When |
|---------|---------|--------|--------------|
| AI Chat | 58-62 | Proactive insight delivery is MVP value; chat deepens engagement after validation | Phase 5 complete, first 10 paying users |
| Insights Detail + Performance | 50-57 | Dashboard (Phase 5) provides Morning Brief; detailed browsing and charts deepen engagement post-core-value | Post-Phase 5 |
| Team Management (invites/roles) | 67-68 | Single-user MVP; RLS infrastructure built from Phase 1 for data isolation | Post-launch, enterprise demand signals |
| Mobile responsive polish | 69-71 | Responsive web as standard practice during Phase 5; dedicated mobile UX optimization deferred | Phase 5 complete |
| Accessibility WCAG 2.1 AA | 72-73 | Tracked as cross-cutting concern (semantic HTML, ARIA labels on new components); formal audit deferred | Post-launch, enterprise/government customers |
| Settings + Billing + Slack | 37, 39-40, 63-66 | Notification preferences, Slack delivery, Stripe billing, and plan management are production-readiness features | Post-Phase 5 |
| Google Ads, Stripe, GA4 integrations | PRD Tier 2/3 | Post-v1 per PRD out-of-scope | Month 2-4 |
| Custom reports/dashboards | — | Post-v1 per PRD out-of-scope | Post-launch, user demand |
| Cohort analysis UI | — | Table exists in schema, no UI per PRD | Post-launch |
| Spanish/bilingual UI | — | Post-$5K MRR per PRD | $5K MRR |

## Parallelizable Phases

- **Phase 6** can start after Phase 2 completes — it does not depend on Phases 3-5 (the data pipeline infrastructure is ready)
- **Settings + Billing** (deferred) can be built in parallel with Phases 4-5 if bandwidth allows

## Risk Notes

- **Phase 2 is the critical path.** Delays here cascade to all downstream phases.
- **Phase 6 (Meta + Klaviyo) involves significant API complexity** (see PRD: Meta API Complexity section). Estimated 3-4 weeks solo for Meta alone. Consider starting Phase 6 immediately after Phase 2 if bandwidth allows.
- **Anomaly detection (Phase 3) with Shopify-only data** produces useful insights (revenue drops, AOV shifts, order volume changes) but cannot diagnose ad performance or email deliverability root causes until Phase 6 lands. This is an accepted trade-off for faster time-to-value.
- **Phase 4 is the highest-risk phase** — it requires LLM integration, email delivery, and the full detection-to-insight-to-email pipeline to work end-to-end for the first time. Allocate buffer here.
- **Phase 3 sub-phases produce no user-facing UI** — the "UI" is stored insight data consumable by Inngest dashboard and API endpoints. Phase 4 (email) provides the first user-facing deliverable. This is intentional: the core engine must work before we build delivery mechanisms.
- **The PRD notes a skipped concierge MVP step.** Phase 4 (first Morning Brief email) is the earliest validation point — if Shopify-only insights don't resonate, the Meta + Klaviyo investment can be re-evaluated.