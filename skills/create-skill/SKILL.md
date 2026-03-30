---
name: create-skill
description: >
  Create a new Claude Code skill and publish it to the product-playbooks repo. Use when the user
  wants to create a new skill, add a playbook, or says "create a skill for [topic]." Handles
  SKILL.md creation, site page generation, index update, and GitHub push.
---

# Create Skill — Skill

## What this does

Creates a new Claude Code skill end-to-end: writes the SKILL.md file, publishes it to the `mixmaxhq/product-playbooks` repo with a documentation page, and installs it locally.

## Workflow

1. **Understand the skill** — ask the user what the skill should do, when it triggers, and what rules/structure it should follow
2. **Write SKILL.md** — create the skill file following the format conventions below
3. **Install locally** — save to `~/.claude/skills/<skill-name>/SKILL.md`
4. **Create site page** — add an `index.html` documentation page under `skills/<skill-name>/` in the product-playbooks repo
5. **Update index** — add a card to the "PM Skills" section in `product-playbooks/index.html`
6. **Push to GitHub** — commit and push to `mixmaxhq/product-playbooks` so it's live on the site

## SKILL.md format conventions

Every skill file follows this structure:

```markdown
---
name: skill-name-in-kebab-case
description: >
  Multi-line description of when to use the skill. Include trigger phrases
  the user might say. Be specific — this description is how Claude decides
  whether to activate the skill.
---

# Skill Title — Skill

## What this does
One paragraph explaining what the skill produces.

## Workflow
Numbered steps — what happens when the skill runs.

## [Domain-specific sections]
Structure, rules, templates, or conventions specific to this skill.
```

## Rules for writing good skills

- **Name** — kebab-case, short, descriptive (e.g. `gtm-brief`, `prd-workshop`, `release-notes`)
- **Description** — this is the trigger. Include exact phrases users might say. Be generous with triggers but specific about what the skill does. Start with the core purpose, then list trigger phrases.
- **Workflow** — numbered steps that describe what the skill does autonomously. Each step should name the tool or action (e.g. "Fetch the Notion page", "Create in Notion under parent page X").
- **Hardcode references** — include specific Notion page IDs, repo URLs, parent pages, and naming conventions so the skill can act without asking. These are the details that make a skill autonomous vs. a generic prompt.
- **Encode lessons learned** — if the skill was born from a manual workflow, capture the corrections and refinements that happened during that workflow (e.g. "don't use internal jargon", "always include customer names"). These are the most valuable parts.
- **Keep it focused** — one skill, one job. Don't combine "create GTM brief" and "create release notes" into one skill.
- **No ephemeral content** — don't include specific dates, ticket numbers, or temporary state. Skills should be durable.

## Site page conventions

The documentation page (`index.html`) for each skill should include these sections in this order:

1. **Title and subtitle** — skill name as h1, one-line description
2. **Install** — curl command to download SKILL.md + link to view on GitHub
3. **How to use** — the slash command with example args
4. **What it does** — one paragraph
5. **Workflow** — numbered steps
6. **[Skill-specific sections]** — structure, rules, examples as relevant
7. **Example** — link to a real output if one exists

Use the existing skill pages in `mixmaxhq/product-playbooks` as format reference.

## Index card format

When adding to `product-playbooks/index.html`, add a card under the "PM Skills" group:

```html
<a class="card" href="skills/<skill-name>/">
  <span class="card-icon">EMOJI</span>
  <div class="card-body">
    <div class="card-title">/<skill-name></div>
    <div class="card-desc">One-line description of what the skill does.</div>
    <div class="card-meta">
      <span class="tag tag-skill">Skill</span>
    </div>
  </div>
  <span class="card-arrow">&rsaquo;</span>
</a>
```

## File locations

- **Local skill:** `~/.claude/skills/<skill-name>/SKILL.md`
- **Repo:** `/Users/jakubtutaj/code/product-playbooks/`
- **Site page:** `product-playbooks/skills/<skill-name>/index.html`
- **Skill file in repo:** `product-playbooks/skills/<skill-name>/SKILL.md`
- **Index:** `product-playbooks/index.html`
- **Site URL:** `https://mixmaxhq.github.io/product-playbooks/`
