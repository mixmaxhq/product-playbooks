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

> **Last updated:** 2026-04-03 — [Published version](https://mixmaxhq.github.io/product-playbooks/skills/amplitude-guide/)

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

**Property name trap:** the field is `productIds` (plural, array type), NOT `productId`. Filtering on `productId` will silently return zero results.

**Gen0 vs gen1 trap (direct-sales contracts):** `productIds = (none)` does NOT reliably mean gen0 (legacy). Direct-sales gen1 contracts like `gen1-copilots-for-teams` often fire `Plan changed` with `productIds` empty AND `plan_title` blank, so they look identical to legacy gen0 in Amplitude. Intercom typically labels these workspaces "Free" because the seats are billed outside the self-serve subscription. To confirm gen0 vs gen1 for a specific user, check the admin tool or billing DB — Amplitude alone is not authoritative.

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

### Scenario: Meeting Copilot analysis
**Ask:** "Are you looking at adoption (meetings recorded), summary engagement, sharing, or bot infrastructure health?"
- **Adoption (best indicator):** Use `MeetingCopilot_Recall_Meeting_Recorded` — the canonical "meeting was successfully recorded" signal. This is the single best indicator of Meeting Copilot adoption: the bot was admitted to the meeting and is actively recording.
- **Summary engagement:** Use `MeetingCopilot_Summary_Accessed` (user viewed summary) and `MeetingAssistant_Followup_Clicked` (user acted on follow-up). These show value delivery.
- **Sharing:** Use `MeetingCopilot_Share_Created` for actual shares. `MeetingCopilot_ShareModal_Opened` is intent only.
- **Bot infrastructure:** Use the BotRunner and Recall event chains (see Meeting Copilot event registry below). Filter `MeetingCopilot_BotRunner_LaunchResult` by `result = "success"` or `"failure"` for launch success rates.
- **Trap:** `MeetingCopilot_BotRunner_LaunchSkipped` has extremely high volume (~1.4M) — it fires for every meeting that gets filtered out. Do NOT use as a proxy for anything user-facing.

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
| `Workspace_Member_Added` | Team member added | Team expansion signal |
| `Tasks` | Tasks feature usage | Filter by `action = "completed task"` for actual task completion — value signal |
| `Rules` | Automation rules | Filter by `action = 'activate'` for successful rule creation — value signal |
| `Calendar_Meeting_Confirmed` | Meeting booked via Mixmax calendar | Value signal — indicates calendar feature delivering outcome |
| `Installed extension` | Extension installation detected | Setup moment indicator |
| `Clicked install extension` | Clicked extension install prompt | Proxy for extension installation — not confirmation of actual install |
| `Viewed message tracking details` | User checks email tracking results in Gmail | Engagement signal — user getting value from tracking |

### Meeting Copilot events (comprehensive registry)

**Best adoption indicator:** `MeetingCopilot_Recall_Meeting_Recorded` — fired when Recall.ai sends a `bot.in_call_recording` webhook. This is the canonical "meeting was successfully recorded" signal: the bot was admitted and is actively recording. Use this as THE metric for Meeting Copilot adoption.

#### Bot lifecycle (infrastructure — use for debugging, not adoption)

| Event | What it means | Key properties |
|-------|--------------|----------------|
| `MeetingCopilot_BotRunner_Invoked` | Bot-runner Lambda passed initial eligibility checks and is evaluating whether to launch. NOT fired for every invocation — early failures go to LaunchSkipped. | ~943K volume |
| `MeetingCopilot_BotRunner_AttemptLaunch` | All eligibility checks passed, about to call Recall.ai API to create bot. The "we decided to launch" signal. | ~12K volume |
| `MeetingCopilot_BotRunner_LaunchResult` | Recall.ai create-bot API call returned. | `result` ("success"/"failure"), `failureCause`, `launchLatencyMs` |
| `MeetingCopilot_BotRunner_LaunchSkipped` | Bot-runner decided NOT to launch. Extremely high volume (~1.4M) — most meetings get filtered out. | `skippedReason` (feature disabled, quota exceeded, bot already exists, user declined, workspace disabled) |
| `MeetingCopilot_BotRunner_Failure` | Unhandled exception crashed the bot-runner Lambda. Not normal skip/failure paths. | `errorType` (validation_error, quota_exceeded_error, recall_ai_error, unknown) |

#### Recording lifecycle (Recall.ai webhooks)

| Event | What it means | Notes |
|-------|--------------|-------|
| `MeetingCopilot_Recall_Bot_Join_Attempt` | Bot is attempting to join the meeting room (bot.joining_call webhook). | |
| `MeetingCopilot_Recall_Bot_In_Waiting_Room` | Bot arrived but host hasn't admitted it yet (bot.in_waiting_room webhook). | |
| `MeetingCopilot_Recall_Waiting_Room_Timeout` | Bot timed out in waiting room — host never admitted it. | Failure signal |
| `MeetingCopilot_Recall_Meeting_Recorded` | **Best adoption indicator.** Bot admitted and actively recording (bot.in_call_recording webhook). | ~4.3K volume. Use this for adoption metrics. |
| `MeetingCopilot_Recall_Call_Ended` | Meeting call ended — bot left or call terminated. | `subCode` (exit code from Recall) |
| `MeetingCopilot_Recall_Bot_Done` | Bot processing fully complete (post-processing finished). | |

#### Summary & value delivery

| Event | What it means | Notes |
|-------|--------------|-------|
| `Meeting_Summarized` | A meeting summary was generated. | ~1.9K volume |
| `MeetingCopilot_Summary_Persisted` | AI summary generated and saved to MongoDB. | `transcriptSize`, `promptVersion`, `meetingClassification` |
| `MeetingCopilot_Summary_Email_Sent` | Summary email delivered to user. | |
| `MeetingCopilot_Summary_Accessed` | User viewed a meeting summary via API. | `access_level` (owner, shared viewer, workspace admin). Value delivery signal. |
| `MeetingAssistant_Followup_Clicked` | User clicked a follow-up action from summary. | Value delivery signal — user acting on summary. |

#### User settings & consent

| Event | What it means | Notes |
|-------|--------------|-------|
| `MeetingAssistant_Settings_FeatureToggled` | User toggled Google Meet or Zoom bot on/off in settings. | `feature` (GoogleMeet_Bot/Zoom_Bot), `status` (Enabled/Disabled) |
| `MeetingAssistant_Settings_SummaryDeliveryChanged` | User changed summary delivery preferences. | |
| `Meeting_Copilot_Consent_Modal_Show` / `_Dismiss` / `_Continue` | Consent modal lifecycle. | Near-dead/dead volume |
| `MeetingCopilot_Consent_Granted` / `_Denied` | User granted or denied consent. | Near-dead volume |

#### Sharing

| Event | What it means | Notes |
|-------|--------------|-------|
| `MeetingCopilot_ShareModal_Opened` | User opened the share modal. | Intent signal |
| `MeetingCopilot_Share_Created` | User actually shared a summary. | Action signal |
| `MeetingCopilot_Share_CopyLink_Clicked` | User copied share link. | |
| `MeetingCopilot_Share_LinkSharing_Toggled` | User toggled link sharing on/off. | |

#### Feedback

| Event | What it means | Notes |
|-------|--------------|-------|
| `MeetingAssistant_Summary_feedback_rated` | User rated a summary (good/meh/bad). | Near-dead volume |
| `MeetingPrep_feedback_rated` | User rated meeting prep. | Low volume |

#### Legacy / Dead Meeting Copilot events (warn or exclude)

| Event | Why it's dead |
|-------|--------------|
| `meet_started` / `meet_ended` | Legacy Chrome extension-based Google Meet bot (pre-Recall.ai). Replaced by `MeetingCopilot_Recall_*` events. |
| `MeetingAssistant_Bot_MeetingLeft` | Legacy self-hosted Zoom bot (pre-Recall.ai). All bot ops now through Recall.ai. |
| `MeetingAssistant_fulltranscript_tab_clicked` / `_visited` | Removed from all codebases. Replaced by `MeetingCopilot_Transcript_Tab_Changed` / `_Visited` in new transcript-webapp. |
| `MeetingAssistant_Summary_Feedback` | Original feedback event (good/bad only). Superseded by `_feedback_rated`. |
| `MeetingAssistant_Summary_feedback_commented` | Enhanced feedback with text comment. Dead volume. |
| `Meeting_Copilot_UpsellCopilotBanner_Viewed` / `_Clicked` | Dead upsell banner events. |

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
