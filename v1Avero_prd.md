# Avero v1 — Product Requirements Document

## Problem Statement

E-commerce brands with $500K–$5M ARR do not have dedicated analysts. When their revenue drops, they open 6 browser tabs (Shopify, Meta Ads, Klaviyo, GA4, Google Ads, Stripe) and spend hours manually cross-referencing data to understand why. They discover problems days too late, miss root causes, and make decisions on gut feeling rather than data. At $99–$299/month, existing tools like Triple Whale ($100–$500/mo) and Northbeam ($250+/mo) offer dashboards and AI chat, but they still require the merchant to ask the right questions. Nobody proactively tells the merchant "your revenue dropped 23% because Meta ROAS fell 40% on campaign X — pause it." That is the gap Avero fills.

## Solution

Avero is an AI-first, semi-agentic analytics platform that **proactively diagnoses why e-commerce metrics change** and delivers actionable recommendations. Instead of dashboards that require interpretation, Avero sends a consolidated daily (or weekly) report that identifies anomalies across Shopify, Meta Ads, and Klaviyo, cross-references root causes, and recommends specific actions. The merchant opens one email or one app and immediately knows what happened and what to do.

**Key differentiators:**

1. **Proactive AI diagnosis** — Avero detects anomalies automatically (no merchant initiation required). It does not wait for questions; it delivers answers.
2. **Root-cause analysis across multiple data sources** — Avero correlates Shopify revenue drops with Meta ROAS declines and Klaviyo deliverability issues in a single narrative.
3. **Consolidated reports, not fragmented alerts** — One report per period, ranked by severity, with all root causes and recommendations in one place.
4. **Semi-agentic** — Avero detects, diagnoses, and recommends. The merchant decides whether to act. v1 does not take actions on behalf of the merchant (no auto-pausing campaigns, no auto-adjusting budgets).
5. **Price-accessible for mid-market DTC** — Starter $79, Pro $149, Growth $299. Targets merchants who cannot afford a $60K/yr analyst.

## User Stories

### Onboarding & Authentication

1. As a merchant, I want to sign up with email/password or Google OAuth, so that I can create my Avero account quickly.
2. As a merchant, I want to start a 7-day free trial without a credit card, so that I can evaluate Avero before committing.
3. As a merchant, I want to select my plan (Starter, Pro, Growth) after the trial, so that I can choose features matching my needs.
4. As a merchant, I want to be prompted to connect Shopify as my first data source, so that I can see value as fast as possible.
5. As a merchant, I want to see a progress indicator while my Shopify data syncs, so that I know Avero is working.
6. As a merchant, I want to receive my first insight within 15–30 minutes of connecting Shopify, so that I experience the "aha moment" quickly.
7. As a merchant, I want to be prompted to connect Meta Ads and Klaviyo after seeing my first insight, so that I can get deeper diagnosis.
8. As a merchant, I want to use Avero with only 1 integration connected, so that I am not blocked if I only have Shopify.
9. As a merchant, I want to see a badge incentivizing me to connect more sources for deeper insights, so that I understand the value of additional connections.

### OAuth & Data Connections

10. As a merchant, I want to connect Shopify via OAuth with clear scopes, so that my store data flows into Avero securely.
11. As a merchant, I want to connect Meta Ads via OAuth, so that my ad performance data is included in diagnosis.
12. As a merchant, I want to select which Meta Ad Account and Pixel to use if I have multiple, so that Avero analyzes the correct data.
13. As a merchant, I want to connect Klaviyo via OAuth, so that my email/SMS performance data is included in diagnosis.
14. As a merchant, I want to see the sync status (last synced, active/error) of each connection in Settings, so that I know my data is fresh.
15. As a merchant, I want to receive an email when a connection expires or errors occur, so that my data does not silently go stale.
16. As a merchant, I want to reconnect a broken connection with one click from Settings, so that I do not lose access to my data.

### Data Sync & Pipeline

17. As the system, I want to sync all active connections every 6 hours via Inngest cron jobs, so that data is fresh for anomaly detection.
18. As the system, I want to perform a full sync (90 days) when a connection is first created, so that historical context exists for baselines.
19. As the system, I want to perform incremental syncs (delta since last_synced_at) on every 6-hour cycle, so that I do not waste API calls on unchanged data.
20. As the system, I want to store raw API payloads in a `raw_data` jsonb column, so that I can reprocess data if an API schema changes without re-fetching.
21. As the system, I want to normalize all timestamps to UTC and all currencies to the merchant's base currency, so that cross-source analysis is accurate.
22. As the system, I want to log every sync job (status, records_synced, errors) in `sync_jobs` and `sync_logs`, so that I can debug failures.
23. As the system, I want to respect Meta API rate limits (~200 calls/hour/endpoint) with throttling and exponential backoff, so that syncs do not fail.

### Anomaly Detection

24. As the system, I want to calculate daily KPI baselines (revenue, orders, AOV, ROAS, MER, email open rate, email revenue) using 30-day rolling averages and standard deviations, so that I can detect anomalies.
25. As the system, I want to flag a KPI as anomalous when its value exceeds 2 standard deviations from the 30-day mean OR shows a >20% WoW change, so that I catch both statistical outliers and meaningful shifts.
26. As the system, I want to run anomaly detection daily at 8:00 AM (Pro/Growth) and weekly on Mondays at 8:00 AM (Starter), so that insights follow the configured frequency.
27. As the system, I want to run critical anomaly detection independently of the daily/weekly schedule (on every sync cycle), so that emergencies like revenue drops >30% in 1 day trigger immediate alerts regardless of tier.

### Insight Generation & LLM

28. As the system, I want to consolidate all detected anomalies into a single report per period (1 per day for Pro/Growth, 1 per week for Starter), so that the merchant receives one clear narrative, not fragmented alerts.
29. As the system, I want to send each consolidated report to the LLM with structured context (last 7 days metrics, all anomalies, connected sources summary), so that the LLM can produce a root-cause analysis.
30. As the system, I want the LLM to return a structured JSON response with `title`, `summary`, `recommendations` (array), `severity`, and `type`, so that insights are consistently parseable and renderable.
31. As the system, I want to assign severity to each report based on the highest-severity anomaly detected (Critical > High > Medium > Low > Normal), so that urgent issues are surfaced first.
32. As the system, I want severity rules to be:
    - **Critical**: Revenue ↓ >30% in 1 day
    - **High**: Revenue ↓ >15% WoW, ROAS ↓ >25% WoW, Email open rate ↓ >40%
    - **Medium**: Revenue ↓ >10% WoW, AOV ↓ >10% WoW
    - **Low**: Any KPI ↓ >10% WoW not meeting higher thresholds
    - **Normal**: All KPIs within expected ranges
33. As the system, I want to store every generated insight in the `insights` table with its full context (anomalies detected, metrics used, recommendations), so that merchants can review historical insights.
34. As the system, I want to track merchant actions on insights (viewed, dismissed, marked as done) in `insight_actions`, so that I can measure insight quality and product value.

### Notifications & Delivery

35. As a merchant on the Starter plan, I want to receive a weekly email on Mondays with my consolidated insight report, so that I stay informed without daily noise.
36. As a merchant on the Pro plan, I want to receive a daily email with my consolidated insight report, so that I can act on issues immediately.
37. As a merchant on the Growth plan, I want to receive daily emails AND Slack notifications, so that I never miss a critical issue.
38. As a merchant, I want to receive an immediate email alert for Critical-severity insights regardless of my plan, so that emergencies are never delayed.
39. As a merchant on the Growth plan, I want to receive an immediate Slack notification for High and Critical severity insights, so that my team sees urgent issues in real time.
40. As a merchant, I want to configure my notification preferences (which severities trigger email vs Slack) in Settings, so that I control my alert volume.
41. As a merchant, I want to see an "All Clear" briefing when no anomalies are detected, so that I know Avero is working and my metrics are healthy.
42. As a merchant, I want each email to include: severity badge, title, root causes, and 2–4 actionable recommendations, so that I can act without opening the app.

### Dashboard — Home (Morning Brief)

43. As a merchant, I want the Home view to show the current Morning Brief (today's consolidated insight report) as the first thing I see, so that I immediately know if something needs attention.
44. As a merchant, I want the Morning Brief to display a severity badge (🔴🟠🟡🟢✅), title, root causes, and numbered recommendations, so that I can understand the situation at a glance.
45. As a merchant, I want a "Ask Avero about this" button on each Morning Brief, so that I can dig deeper into the insight via AI chat with pre-loaded context.
46. As a merchant, I want 4 Quick Metrics cards (Revenue, Orders, AOV, MER) with delta percentages below the Morning Brief, so that I see the headline numbers immediately.
47. As a merchant, I want a 7-day revenue trend chart (Tremor AreaChart) below Quick Metrics, so that I can see the visual pattern behind the numbers.
48. As a merchant, I want a Channel Breakdown section with cards for Shopify, Meta Ads, and Klaviyo showing key metrics per channel, so that I see which channel is driving the change.
49. As a merchant, I want a Recent Insights list (last 5) with severity badges and timestamps below the chart, so that I can scan recent history without navigating away.

### Dashboard — Insights

50. As a merchant, I want an Insights list view with filter tabs (All, Critical, High, Medium, Low, Normal), so that I can find specific types of insights.
51. As a merchant, I want each insight card in the list to show severity badge, date, title, and a one-line summary, so that I can scan quickly.
52. As a merchant, I want to click "View details" on an insight to see: full summary, root causes with data, numbered recommendations with estimated impact, and underlying charts, so that I can understand the full picture.
53. As a merchant, I want to mark each recommendation as "Done" or "Dismiss" on the insight detail page, so that I can track which actions I have taken.
54. As a merchant, I want an "Ask Avero" button on each insight detail page, so that I can ask follow-up questions about that specific insight.

### Dashboard — Performance

55. As a merchant, I want a Performance view with a date range selector and a comparison toggle (vs previous period), so that I can explore trends.
56. As a merchant, I want the Performance view to show: Overview metrics, Revenue trend chart, Channel performance table, Top products table, and Ad campaigns table (if Meta connected), so that I can drill into any dimension.
57. As a merchant, I want an "Ask Avero about this data" button on the Performance view, so that I can ask questions about specific data points.

### Dashboard — AI Chat

58. As a merchant, I want to open an AI chat panel (slide-out on desktop, full view on mobile) from any page, so that I can ask questions contextually.
59. As a merchant, I want the AI chat to have access to my last 30 days of metrics and recent insights as context, so that answers are grounded in my actual data.
60. As a merchant, I want the AI chat to suggest 3 prompted questions below the input field, so that I can ask common questions without typing.
61. As a merchant, I want the AI chat to stream responses in real time, so that I see the answer building up instead of waiting.
62. As a merchant, I want the AI chat to reference specific numbers and trends from my data in its answers, so that I trust the diagnosis.

### Dashboard — Settings

63. As a merchant, I want to see all my connected data sources with status (connected, syncing, error, expired) and last synced timestamp, so that I know my data is current.
64. As a merchant, I want to connect additional data sources (Google Ads, Stripe, GA4) based on my plan tier, so that I can expand my diagnosis depth.
65. As a merchant, I want to configure notification preferences (email severities, Slack severities), so that I control my alert volume.
66. As a merchant, I want to manage my subscription plan and billing via Stripe, so that I can upgrade or cancel.
67. As a merchant, I want to invite team members with roles (owner, admin, viewer), so that my team can access Avero.
68. As an owner, I want Row Level Security to ensure every team member can only see data belonging to our tenant, so that data isolation is enforced at the database level.

### Mobile Experience

69. As a merchant, I want a fully responsive mobile layout with the same priority (Morning Brief first), so that I can check Avero on my phone.
70. As a merchant, I want AI Chat to be a full-screen view on mobile, so that I have enough space to read responses.
71. As a merchant, I want to swipe through Quick Metrics on mobile, so that I can see all 4 KPIs without scrolling.

### Accessibility & Internationalization

72. As a merchant, I want the UI language to be English (v1), so that the product targets the US/UK/CA market first.
73. As a merchant, I want the UI to be fully accessible (keyboard navigation, screen reader labels, sufficient color contrast), so that Avero meets WCAG 2.1 AA standards.

## Implementation Decisions

### Product Positioning & Business Model

- **Name**: Avero
- **Target ICP**: DTC e-commerce merchants, $500K–$5M ARR, Shopify-based, 1–10 person team, founder does marketing. No dedicated analyst.
- **Geographic focus**: US/UK/CA (English-only v1). LATAM/EU bilingual support deferred to post-$5K MRR.
- **Pricing**: Starter $79/mo (1 integration, weekly insights, no AI chat), Pro $149/mo (3 integrations, daily insights, AI chat), Growth $299/mo (all integrations, daily insights, AI chat, Slack alerts). Annual discount 20%.
- **Free trial**: 7 days, no credit card required.
- **Differentiator**: Proactive AI diagnosis across multiple data sources, not dashboards.

### Data Sources (Integration Tiers)

- **Tier 1 (v1, day 1)**: Shopify, Meta Ads, Klaviyo
- **Tier 2 (month 2–3)**: Google Ads, Stripe
- **Tier 3 (month 4+)**: GA4, TikTok Ads, Pinterest, Snap, Microsoft Ads

### Tech Stack

- **Frontend**: Next.js 14 (App Router, Server Components), TypeScript (strict), Tailwind CSS, shadcn/ui, Tremor (charts)
- **Backend**: Next.js API Routes, tRPC (type-safe API), Zod (validation)
- **Database**: Supabase (Postgres with Row Level Security, Supabase Auth, Supabase Storage)
- **Background Jobs**: Inngest (serverless job queue with retries, rate limiting, dashboard)
- **Integrations**: Shopify (`@shopify/shopify-api`), Meta (`facebook-nodejs-business-sdk`), Klaviyo (REST API)
- **AI**: OpenAI SDK + Vercel AI SDK (streaming), GPT-4o-mini for v1 (upgrade to GPT-4o or OSS model post-$10K MRR)
- **Email**: Resend (templates + transactional)
- **Payments**: Stripe (subscriptions + billing)
- **Hosting**: Vercel (Next.js), Supabase Cloud (DB)
- **Monitoring**: Sentry (errors), LogSnag (logs + metrics), PostHog (product analytics)

### Database Schema (Supabase Postgres with RLS)

All tables use `bigint GENERATED ALWAYS AS IDENTITY` primary keys (not UUIDv4, to avoid index fragmentation). All identifiers use `snake_case` lowercase. All tables have RLS enabled with policies using `(SELECT auth.uid())` for performance (not bare `auth.uid()` per-row). Complex tenant access checks use a `SECURITY DEFINER` function `private.user_has_tenant_access(tenant_id bigint)`.

**Core tables**: `tenants`, `users`, `memberships` (multi-tenant with RLS)

**Connections**: `connections` (OAuth tokens stored encrypted via Supabase Vault, metadata as jsonb)

**Raw data** (partitioned by `created_at` using Postgres range partitioning): `shopify_orders`, `shopify_products`, `shopify_customers`, `meta_ad_accounts`, `meta_campaigns`, `meta_insights`, `klaviyo_profiles`, `klaviyo_campaigns`, `klaviyo_events`. Each includes a `raw_data jsonb` column for full API payload preservation and reprocessing capability.

**Analytics**: `daily_metrics` (partitioned, aggregated daily per tenant: revenue, orders, AOV, ad_spend, impressions, clicks, ROAS, email metrics, MER), `channel_metrics`, `customer_cohorts`, `product_performance`

**AI/Insights**: `insights` (type, severity, title, summary, recommendations as jsonb, context as jsonb), `insight_actions` (viewed, dismissed, acted_on tracking), `chat_sessions`, `chat_messages`

**Jobs**: `sync_jobs`, `sync_logs`

All foreign key columns are indexed. Tenant isolation is enforced via RLS policies that call `private.user_has_tenant_access()`.

### Anomaly Detection Logic

Anomalies are detected using two rules (any match triggers):
1. **Statistical**: Value exceeds 2 standard deviations from 30-day rolling mean
2. **Threshold**: WoW change exceeds 20%

Severity assignment uses cascading rules (highest severity wins):
- **Critical**: Revenue ↓ >30% in 1 day
- **High**: Revenue ↓ >15% WoW, ROAS ↓ >25% WoW, Email open rate ↓ >40%
- **Medium**: Revenue ↓ >10% WoW, AOV ↓ >10% WoW
- **Low**: Any KPI ↓ >10% WoW not meeting higher thresholds
- **Normal**: All KPIs within expected ranges

### LLM Prompt Architecture

The LLM receives a structured JSON context containing: tenant ID, detected anomalies (metric, direction, magnitude, WoW context), last 7 days of metrics (revenue, orders, AOV, ad spend, ROAS, MER, email metrics), and connected sources summary. It returns a structured JSON response with: `title` (max 50 chars), `summary` (2–3 sentences), `recommendations` (array of 2–4 strings with estimated impact), `severity` (enum), and `type` (enum). Temperature is set to 0.3 for consistency.

### Notification Frequency

- **Data sync**: Every 6 hours for all tiers
- **Insight generation**: Daily at 8:00 AM (Pro/Growth), Weekly Monday 8:00 AM (Starter)
- **Critical alerts**: Immediate email (all tiers), immediate Slack (Growth only)
- **High alerts**: Included in daily/weekly email (Pro/Growth), included in weekly email (Starter)
- **Normal (all clear)**: Included in daily/weekly briefing (all tiers)

### Report Format

Each period produces exactly **1 consolidated report** (never multiple fragmented alerts). The report combines all detected anomalies into a single narrative with root-cause analysis and prioritized recommendations. Report severity equals the highest-severity anomaly detected.

### Onboarding Flow

1. Signup (email/password or Google OAuth) via Supabase Auth → create tenant
2. Plan selection (Starter/Pro/Growth) → 7-day trial starts (no card required)
3. Connect Shopify (OAuth, scopes: `read_orders, read_products, read_customers, read_analytics`)
4. Full sync (90 days) runs via Inngest, progress shown in UI
5. First insight generated and displayed within 15–30 minutes
6. Prompt to connect Meta Ads and Klaviyo for deeper insights
7. Recommendations can be acted on (mark as done/dismiss) or explored via AI chat

Error handling: toast notifications (shadcn/ui) for UI errors, email alerts for critical sync/connection failures, automatic retry with exponential backoff for API failures.

### AI Chat Architecture

- Slide-out panel on desktop, full-screen view on mobile
- Uses Vercel AI SDK for streaming responses
- System prompt includes last 30 days of `daily_metrics` and last 5 `insights` as context
- Suggested questions rendered below input field
- Chat history stored in `chat_sessions` + `chat_messages` tables

### UI Components (Tremor + shadcn/ui)

Card (metrics, insights, channel breakdown), AreaChart (revenue trend, ROAS trend), BarChart (top products, ad campaigns), Table (channel performance, campaign list), Badge (severity), Button, Callout (Morning Brief), Delta (% changes), TextInput (chat), Dialog (insight detail), Toast (errors + notifications), TabList/TabGroup (insight filters).

### Estimated Monthly Cost at 30 Pro Customers

- Vercel: $20 (Pro plan)
- Supabase: $25 (Pro plan)
- Inngest: Free (under 25K jobs/month)
- Resend: $20
- OpenAI: $60–90 (GPT-4o-mini for daily insights + chat)
- Sentry: Free (under 5K errors)
- LogSnag: $29 (Pro plan)
- Stripe fees: ~$130 (2.9% + $0.30 × 30 × $149)
- **Total: ~$284–414/mes** against $4,470 MRR → ~$4,100 gross margin

## Testing Decisions

### What Makes a Good Test

- Test external behavior, not implementation details. A test should verify that given a set of metrics, the anomaly detection engine flags the correct KPIs with the correct severity. It should not assert internal function call order.
- Test at the highest possible seam. Prefer testing the Inngest function that generates insights over testing the SQL query that calculates WoW change. Prefer testing the API response that returns insights over testing the LLM prompt.
- Mock external dependencies (Shopify API, Meta API, Klaviyo API, OpenAI API) at the network level.
- Use real Supabase instances (Supabase local dev or test project) for integration tests, not mocks of the database.

### Modules to Test

1. **Anomaly detection engine** — Given a set of `daily_metrics`, verify correct anomaly flags (metric, direction, magnitude, severity). Test edge cases: single data point, 0 revenue days, all-normal periods, cascading anomalies.
2. **Insight generation pipeline** — Given anomalies and context, verify the LLM produces parseable JSON with all required fields. Test with mocked LLM responses.
3. **Sync pipeline** — Given a connection, verify that Shopify/Meta/Klaviyo data is fetched, transformed, and stored correctly. Test with API mocks.
4. **RLS policies** — Given two tenants, verify that tenant A cannot read tenant B's data. Test with SQL assertions.
5. **Notification delivery** — Given an insight with severity High, verify that an email is sent with correct content. Test with Resend mock.
6. **AI chat** — Given a tenant context and a user question, verify that the response includes relevant data references. Test with Vercel AI SDK mock.

### Prior Art

No prior tests exist (greenfield project). The test structure will follow Next.js conventions: `__tests__/` directories alongside source files, Vitest for unit tests, Playwright for E2E tests.

## Out of Scope

### v1 Does Not Include

- **Google Ads, Stripe, GA4, TikTok Ads integrations** — Tier 2 and 3 integrations are deferred to months 2–4.
- **Automated actions** — Avero will not pause campaigns, adjust budgets, send emails, or modify any merchant data. All actions are recommendations; the merchant decides.
- **Full agentic mode** — Avero detects and diagnoses, but does not act autonomously. v2+ may include auto-pause, auto-budget-reallocate with merchant approval.
- **Real-time data** — Data syncs every 6 hours only. Meta attribution data has 24–48h lag regardless. Real-time sync is theater.
- **Custom reports/dashboards** — v1 has fixed views (Home, Insights, Performance, Chat, Settings). No report builder, no custom widgets.
- **Team collaboration** — v1 supports multiple team members per tenant, but no comments, mentions, or shared boards.
- **Mobile native app** — v1 is responsive web only. No iOS/Android app.
- **White label / agency mode** — v1 is single-tenant per account. No agency dashboard managing multiple merchants.
- **Bilingual/Spanish UI** — v1 is English only. Spanish UI deferred to post-$5K MRR.
- **Attribution modeling** — Avero does not build its own attribution model. It reports each source's own attribution (Meta ROAS is Meta's number, Shopify revenue is Shopify's number). Discrepancies are noted, not resolved.
- **Forecasting / predictive** — v1 detects past and present anomalies. It does not predict future revenue.
- **Cohort analysis UI** — `customer_cohorts` table exists in the schema for future use, but no cohort UI in v1.
- **Concierge/white-glove onboarding** — v1 is self-serve OAuth only. No manual data connection.

## Further Notes

### Meta API Complexity

Meta Ads integration is the most complex integration in v1 due to: (1) two separate APIs (Marketing API + Conversions API/CAPI), (2) iOS 14.5+ attribution changes requiring AEM and domain verification, (3) aggressive rate limits (~200 calls/hour/endpoint), (4) 24–48h data lag on attribution metrics, (5) multiple attribution windows (1d/7d/28d click, 1d/7d/28d view), (6) frequent API version deprecations, (7) inevitable discrepancies vs Shopify numbers due to refunds, timezones, and attribution models. Estimated integration time: 3–4 weeks solo full-time.

### Data Discrepancy Handling

Avero will NOT attempt to reconcile discrepancies between sources. Instead, each insight will note the source of each metric: "Meta reports $X in purchases; Shopify reports $Y." The Morning Brief will include a footnote when discrepancies exceed 10%. "Why don't the numbers match?" is a support FAQ, not a v1 engineering problem.

### LLM Model Flexibility

The LLM abstraction (in `lib/ai/`) uses an interface that can swap between OpenAI, Anthropic, and OSS models. v1 ships with GPT-4o-mini for cost reasons ($0.15/1M input tokens). The architecture supports swapping to GPT-4o, Claude, or a self-hosted model post-validation without changing business logic. LLM model selection is a config parameter, not a code change.

### Concierge MVP Note

The original recommendation was to validate with a manual concierge MVP (5–10 merchants, manual data analysis via Claude/ChatGPT, weekly email brief) before building the product. The founder chose to skip this step and build directly. This decision carries risk: without prior validation, the product may discover in month 3 that the ICP wants different insights, different delivery, or a different pricing model. This risk is acknowledged and accepted.

### Name and Trademark

"Avero" — derived from Latin "averiguare" (to find out / to verify). Before committing to the name, verify trademark availability at uspto.gov, euipo.europa.eu, and Shopify App Store existing apps. Also verify domain availability (avero.com, avero.ai, getavero.com).