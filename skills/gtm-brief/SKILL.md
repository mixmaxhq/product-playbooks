---
name: gtm-brief
description: >
  Create a product brief for GTM (go-to-market) from a PRD or feature spec. Use when the user
  asks to write a GTM brief, product brief, sales enablement doc, or wants to prepare a feature
  for the GTM team. Trigger: "GTM brief", "product brief", "brief for sales", "brief for GTM",
  or when a PRD needs to be translated into customer-facing documentation.
---

# GTM Product Brief — Skill

## What this does

Takes a PRD or feature description and produces a structured product brief for the GTM team (sales, CS, marketing). The brief lives as a Notion page under the [Product briefs for GTM Repo](https://www.notion.so/mixmax/29c7507ac62280ada9b1c1c7189d3df1).

## Workflow

1. **Fetch the source PRD** — user provides a Notion URL or describes the feature
2. **Fetch an existing GTM brief** from the repo for format reference (use the most recent one)
3. **Generate the brief** following the structure below
4. **Create the Notion page** under the GTM Repo parent page (`29c7507ac62280ada9b1c1c7189d3df1`)
5. **Link back** — add a callout at the top of the original PRD linking to the new brief
6. **Review with user** — iterate on feedback before finalizing

## Brief structure

```
## Feature Name & Eligibility
- Feature (user-facing name)
- Eligible plans (and explicitly note which plans it's NOT on, if relevant for upsell/renewal)
- Rollout approach
- Target release

---
## Value Proposition
One paragraph — what it does for the customer, in plain language.

---
## Problems We Solve
2-3 named problems with short descriptions. Include customer names who requested it if known.

---
## How We Solve It
Subsections for each major capability. Plain language, no engineering jargon.

---
## How It Works
Numbered steps — the user journey from enabling to getting value.

---
## FAQ
Q&A format — anticipate what sales/CS will be asked by customers.

---
## GTM Talking Points
3 numbered talking points with quotable one-liners.

---
## Competitive Positioning
How this compares to alternatives. Keep it short.

---
## Suggested Customer Messaging
- CSM outreach template (email or Slack)
- Discovery questions for prospects
- Differentiation statement

---
## GTM Collateral Checklist
Table: Collateral | Description | Needed? (Y/N)
```

## Translation rules

These rules ensure the brief reads well for salespeople, not engineers:

- **No internal jargon** — Replace "PLG flows" with "built-in invite surfaces" or "onboarding screens." Replace "prepaidAccounts" with "pending member records." No acronyms without explanation.
- **Use the customer-facing feature name** — not the internal project name. The PRD title and the GTM brief feature name may differ.
- **Frame for the customer, not the codebase** — "admins get full control" not "backend guard on prepaidAccounts insertion." Benefits over implementation.
- **Say "enterprise customers" or "customers with large sales teams"** — not "Direct Sales customers" (that's our internal segment name).
- **Include customer names** — if specific customers requested the feature, name them in Problems We Solve and the collateral checklist for targeted outreach.
- **Self-serve when possible** — if the feature can be enabled by the customer themselves, direct them to the settings URL rather than "reach out to us."
- **Renewal/upsell framing** — if the feature is exclusive to certain plans, explicitly call out which plans DON'T have it and frame as a renewal/upsell lever.
- **No p-values or statistical language** — keep it business-level.
- **Pyramid principle** — lead with the takeaway, details below.
