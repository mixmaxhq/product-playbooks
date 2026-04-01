---
name: amplitude-guide
description: >
  Guide for analyzing Mixmax product data using Amplitude MCP. Use when the user
  asks about self-serve traffic, conversion to paid, activation funnels, aha moments,
  signup trends, pricing page behavior, or any product analytics question.
  Trigger: "analyze signups", "check conversion", "how many users paid",
  "self-serve funnel", "activation rate", "what events should I use",
  "is this event reliable", "run this analysis", "check this report",
  "amplitude analysis", "data question".
---

# Amplitude Guide — Skill

## What this does
Guides data analysis on Mixmax Amplitude data by ensuring the right events are used with correct filters. Prevents common traps (wrong event, missing filter, misleading metric). Can run analysis via Amplitude MCP tools or review existing charts.

## Before any analysis — always ask:

1. **Timeframe** — What date range are you analyzing?
2. **Conversion window** — Which window applies? (1-day, 14-day, 15-day, 30-day, 60-day, 90-day)
3. **Scenario-specific filters** — Based on the question, ask the relevant filter questions (see Filter Scenarios below)

## Amplitude MCP connection
- **Project ID:** 130895
- Always call `get_context` at start of session to confirm access
- Use `query_chart` for deep-diving existing charts, `query_charts` for comparing multiple
- Use `query_dataset` for custom funnels and event segmentation

---

## Universal filters (always apply)

These filters apply to EVERY analysis, no exceptions:

| Filter | How to apply | Why |
|--------|-------------|-----|
| Exclude Mixmax Test Users | `userdata_cohort is not vbyym9zo` | Test accounts pollute all metrics |
| Exclude auto-connected integrations | On `Connected Third-Party Integration`: exclude `google` and `microsoft` | These are authentication methods, not real integrations |

---

## Filter scenarios

When you detect the user's question matches a scenario, ask the relevant filter questions:

### Scenario: Self-serve acquisition funnel
**Ask:** "Are you looking at first-admins only, or all signups including invited users?"
- First-admins only: filter `Signup (server)` by `isFirstWorkspaceAdmin = True`
- All signups: no filter, but warn that Virality leadSource includes workspace invites — segment with `viralSource` property if needed

### Scenario: Conversion to paid
**Ask:** "Are you including free users or paid-only? Are you including direct sales or self-serve only?"
- Free included: use `Plan changed` without productId filter
- Paid only: filter `Plan changed` by productId — only these count as paid purchases:
  - `[gen1-inbox-copilot]`
  - `[gen1-engagement-copilot]`
  - `[gen1-custom-branding-addon, gen1-inbox-copilot]`
  - `[gen1-custom-branding-addon, gen1-engagement-copilot]`
  - `[gen1-bundle, gen1-custom-branding-addon]`
  - `[gen1-engagement-copilot, gen1-inbox-copilot]`
  - `[gen1-custom-branding-addon, gen1-engagement-copilot, gen1-inbox-copilot]`
  - `[gen1-meeting-copilot]`
  - `[gen1-bundle]`
  - `[gen1-custom-branding-addon, gen1-engagement-copilot, gen1-meeting-copilot]`
  - Legacy: `mixmaxEnterpriseAnnualOct2017`, `mixmaxProfessionalAnnualOct2017`, `mixmaxProfessionalMonthlyOct2017`
- Exclude trials: filter out `[gen1-bundle-trial]`, `[gen1-inbox-copilot-trial]`
- Exclude free downgrades: filter out `[gen1-free]`, `Free`, `[]` (empty)
- Exclude direct sales: filter out `[gen1-copilots-for-teams, gen1-custom-branding-addon]`

### Scenario: Email activity analysis
**Ask:** "What type of email activity are you interested in? Sequence emails, template emails, emails with calendar enhancements, or emails triggered by insights (Todo/Smart Follow-up)?"
- Then apply the POSITIVE filter for what they want on `Sent email on server`:
  - Sequence emails: `withSequence = true`
  - Template emails: `withTemplate = true`
  - Calendar-enhanced emails: `calendarEnhancementTypes` contains `calendarlink`, `availability`, `event`, or `groupscheduling`. **Trap:** `default_calendarlink_in_signature` alone is passive/auto-added — NOT a real calendar enhancement
  - Insight-driven emails: `relatedMixmaxInsights` is not `[]`
- **Do NOT** analyze `Sent email on server` without a specific filter — plain emails with no Mixmax features are noise

### Scenario: Traffic source analysis
**Ask:** "Are you looking at first-admins only? Do you want to segment by lead source?"
- Warn: `leadSource = Virality` includes workspace invites when not filtered to `isFirstWorkspaceAdmin`
- Can segment further using `viralSource` property
- `jobRole` segmentation is always available and useful for ICP analysis

### Scenario: Activation / aha moment analysis
**Ask:** "Which aha moment? Sequence activation, email with calendar, or action taken (Todo/Smart Follow-up)?"
- Aha #1 (Sequence activation): `SequenceEditor_ConfirmActivateSequence_Clicked` or `SequenceServer_ActivateRecipients_Activated` — most correlated with payment (31% conversion rate)
- Aha #2 (Action taken): available only from Feb 18, 2026 onward (event instrumented that date)
- Aha #3 (Email with calendar): see calendar filter trap above

---

## Event registry

### Tier 1: Core funnel events (reliable, curated)

| Event | What it means | Key properties |
|-------|--------------|----------------|
| `Signup (server)` | User signed up | `isFirstWorkspaceAdmin`, `leadSource`, `viralSource`, `jobRole`, `experimentVariant`, `signupPrimaryService` |
| `Plan changed` | Payment or plan change | `productId` (see filter scenarios for valid paid values) |
| `ShoppingCart_ProductComparison_PageLoaded` | Viewed pricing/comparison page | Critical early touchpoint — 29% of buyers hit this before any feature |
| `ShoppingCart_Summary_PageLoaded` | Viewed checkout summary | Purchase intent signal |
| `ShoppingCart_PayNow_Clicked` | Clicked pay button | Direct purchase intent |
| `Billing - purchase` / `Billing - purchased for` | Purchase confirmed | Final conversion event |

### Tier 2: Activation & feature events (curated, need specific filters)

| Event | What it means | Traps |
|-------|--------------|-------|
| `Sent email on server` | Email sent through Mixmax | MUST use specific filters (see Email scenario). Never analyze unfiltered. |
| `SequenceEditor_ConfirmActivateSequence_Clicked` | User activated a sequence | Aha #1 — strongest payment predictor |
| `SequenceServer_ActivateRecipients_Activated` | Server-side sequence activation | Pair with above for validation |
| `Sequences_Sequence_Created` | Created a new sequence | Creation ≠ activation — don't confuse |
| `Connected Third-Party Integration` | Connected a 3rd-party service | Exclude google/microsoft (universal filter). Signal: linkedin, salesforce, hubspot, zoom |
| `App_Navigation_PageLoaded` | Page navigation | Properties: `page`, `subpage`. Useful for journey analysis, NOT conversion |
| `Template` | Used email template | Filter by `action = "create"` for actual template creation. Without filter, includes views/uses too. |
| `meeting templates` | Calendar setup completed (when filtered by `action = updated`) | MUST filter by `action = updated` — other action values are not calendar setup completion |
| `MeetingCopilot_Summary_Attempt` / `_Accessed` / `_SharedEmail_Sent` | Meeting Copilot usage | Low volume, niche feature |
| `Workspace_Member_Added` | Team member added | Team expansion signal |
| `Tasks` | Tasks feature usage | Filter by `action = "completed task"` for actual task completion — value signal |
| `Rules` | Automation rules | Filter by `action = 'activate'` for successful rule creation — value signal |
| `Calendar_Meeting_Confirmed` | Meeting booked via Mixmax calendar | Value signal — indicates calendar feature delivering outcome |
| `Installed extension` | Extension installation detected | Setup moment indicator |
| `Clicked install extension` | Clicked extension install prompt | Proxy for extension installation — not confirmation of actual install |
| `Viewed message tracking details` | User checks email tracking results in Gmail | Engagement signal — user getting value from tracking |

### Tier 3: Noise events (warn or exclude)

If the user tries to use any of these, warn them:

| Event | Why it's noise |
|-------|---------------|
| `Onboarding_Step_Shown` / `_Finished` | Onboarding instrumentation, not product usage |
| `OnboardingHub_*` | Onboarding hub events |
| `Dashboard onboarding finished` | Onboarding completion marker, not activation |
| `Created profile` | Profile creation during onboarding. Exception: when `hasExtensionInstalled = true`, signals setup moment |
| `Created workspace` | Workspace creation, not meaningful product action |
| `Message sent via Mixmax Lite` | Lite users, different product surface |
| `Calendar inserted` | Passive action, not meaningful |
| `User_Levers_Changed` | Internal lever system |
| `Sent email on server` (unfiltered) | Without feature filters, this is just "sent any email" — meaningless |
| `Shared conversations` / `Sent sidechat message` | Alias for same event. Filter by `action = "added comment"` for active Sidechat usage — without filter it's noise |
| Gmail UI performance events | System telemetry |
| `LayoutMigration_*` | Internal migration tracking |
| `ImportContacts_*` | Contact import system events |

---

## Curated starting-point charts

These are pre-built, correctly-filtered Amplitude charts. Use them as starting points — clone and adjust timeframe/filters as needed:

### Funnel charts
| Chart | What it shows | ID |
|-------|-------------|-----|
| [Signup cohort count](https://app.amplitude.com/analytics/mixmax/chart/vmbij3nh) | Total first-admin signups | `vmbij3nh` |
| [Signup → Setup moment](https://app.amplitude.com/analytics/mixmax/chart/fbjllgy3) | Setup completion, 1-day window | `fbjllgy3` |
| [Signup → Aha: sequence activation](https://app.amplitude.com/analytics/mixmax/chart/wr5emvlc) | Sequence activation, 14-day window | `wr5emvlc` |
| [Signup → Aha: email with calendar](https://app.amplitude.com/analytics/mixmax/chart/xsbdy38i) | Calendar email aha, 14-day window | `xsbdy38i` |
| [Signup → Aha: action taken](https://app.amplitude.com/analytics/mixmax/chart/ta7fai08) | Action taken aha, 14-day window (Feb 18+ only) | `ta7fai08` |
| [Signup → Paid (Gen1)](https://app.amplitude.com/analytics/mixmax/chart/0k4fn8ng) | Full conversion funnel, 30-day window | `0k4fn8ng` |
| [Signup → Sequences page → Sequence sent](https://app.amplitude.com/analytics/mixmax/chart/rlbd1zza) | Sequence visit-to-send funnel, 14-day window | `rlbd1zza` |

### Trending charts
| Chart | What it shows | ID |
|-------|-------------|-----|
| [Funnel conversion rates (weekly)](https://app.amplitude.com/analytics/mixmax/chart/jtj0q3wl) | Signup → Purchase weekly rate | `jtj0q3wl` |
| [Funnel volumes (weekly)](https://app.amplitude.com/analytics/mixmax/chart/dl01qjei) | Absolute funnel counts weekly | `dl01qjei` |

### Traffic charts
| Chart | What it shows | ID |
|-------|-------------|-----|
| [Signups by leadSource (weekly)](https://app.amplitude.com/analytics/mixmax/chart/zcss1n1r) | Traffic mix breakdown | `zcss1n1r` |
| [Best-converting source signups (monthly)](https://app.amplitude.com/analytics/mixmax/chart/h4sr1u19) | High-quality source trends | `h4sr1u19` |
| [Total new workspace signups (monthly)](https://app.amplitude.com/analytics/mixmax/chart/n5gilxlz) | Total signup volume | `n5gilxlz` |

### Correlation charts
| Chart | What it shows | ID |
|-------|-------------|-----|
| [Moments to conversion correlation](https://app.amplitude.com/analytics/mixmax/chart/pcz5yusg) | Aha moments vs payment correlation | `pcz5yusg` |

---

## Common analysis recipes

### "What's our signup to paid conversion rate?"
1. Ask: timeframe, conversion window, free vs paid, include direct sales?
2. Start with chart `0k4fn8ng` (Signup → Paid, 30-day window) — adjust window as needed
3. Filter `Plan changed` by productId for paid-only
4. Segment by `leadSource` or `jobRole` if they want breakdowns

### "How are signups trending?"
1. Ask: timeframe, first-admins only?
2. Start with chart `n5gilxlz` (total monthly) or `zcss1n1r` (by leadSource weekly)
3. Warn: total signup numbers are misleading because they include non-performing viral signups (0.1% CVR). The metric that matters is Google Organic trajectory.

### "Which aha moments predict payment?"
1. Start with chart `pcz5yusg` (moments to conversion correlation)
2. Key finding: Sequence activation has 31% correlation with payment — highest-leverage aha moment
3. Setup moment without sequence activation converts at ~1.85%
4. No setup → 0% conversion

### "How is the self-serve funnel performing?"
1. Ask: timeframe, conversion window
2. Use charts `jtj0q3wl` (rates) and `dl01qjei` (volumes) together
3. Warn: rate can be misleading when traffic mix shifts. Always check absolute volumes alongside rates.
4. Warn: recent weeks may have incomplete conversion windows — note which cohorts have closed windows
