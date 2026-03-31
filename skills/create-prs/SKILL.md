---
name: create-prs
description: >
  Pre-flight checks and best practices for creating GitHub PRs in Mixmax repos.
  Use when creating a PR, preparing code for review, or before pushing changes.
  Trigger: "create PR", "prepare PR", "push and create PR", "pre-flight checks",
  "check before PR", "ready for review".
---

# Creating PRs — Skill

## What this does
Runs pre-flight checks on your code changes, fixes common CI failures, and creates clean PRs with proper descriptions. Encodes lessons learned from real Mixmax CI pipelines to catch issues before they reach GitHub Actions.

## Workflow

1. **Run pre-flight checks** on all changed files (see checklist below)
2. **Fix any issues found** — lint errors, formatting, type errors
3. **Verify tests pass** locally before pushing
4. **Squash commits** if needed into clean conventional commit(s)
5. **Push and create PR** with proper description, links, and test plan
6. **Cross-link** companion PRs if changes span multiple repos

## Pre-flight checklist

Before creating a PR, run these checks on every changed file:

### 1. Lint
```bash
npx eslint <changed-files>
```
Common errors to watch for:
- `react/jsx-fragments` — use `<React.Fragment>` not `<>` shorthand
- `no-console` — don't commit files with `console.log` (especially scripts/tools)
- `prettier/prettier` — formatting issues, run `npx prettier --write <file>` to fix
- `object-shorthand` — use `{ userId }` not `{ userId: userId }`
- `import/order` — imports must be in the correct group order

### 2. Prettier
```bash
npx prettier --write <changed-files>
```
Always run Prettier before committing. CI will fail on formatting issues even if ESLint passes locally with a different Prettier version.

### 3. TypeScript
```bash
npx tsc --noEmit
```
Watch for:
- `TS2345` — type mismatches, especially with array element access (use `!` non-null assertion when you know the array is non-empty)

### 4. Tests
```bash
npx jest --testPathPatterns="<test-file>" --no-coverage
```
Run tests for every file you changed or added. Verify both new and existing tests pass.

### 5. Commit messages
Mixmax repos use [Conventional Commits](https://www.conventionalcommits.org/) with commitlint.

**Critical rule: scope must be lowercase.**
- `feat(pg-963)` — correct
- `feat(PG-963)` — WILL FAIL commitlint (`scope-case: lower-case`)

Format:
```
type(scope): short description

Optional body explaining why, not what.

Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>
```

Types: `feat`, `fix`, `chore`, `style`, `docs`, `test`, `refactor`

### 6. Dev-only code
Before creating a PR, remove:
- Local helper scripts (they fail `no-console` lint)
- Query param overrides for local testing
- Hardcoded test values or debug code
- Any file that should stay untracked

If dev tooling is useful to keep, add it to `.gitignore` or keep it outside the repo.

## Multi-repo PRs

When changes span multiple repos (e.g., `app` + `source-tracking`):

1. **Create the simpler/dependency PR first** — e.g., the tracking PR before the UI PR
2. **Note "companion PR to follow"** in the first PR description
3. **Create the second PR** and link back to the first using `owner/repo#number` format
4. **Update the first PR** description to cross-link: `Companion PR: mixmaxhq/app#16281`

## PR description template

```markdown
## Summary
- 1-3 bullets explaining what and why (not how)

## Key changes
- Bullet per significant change area

## Links
- Spec: [Notion page title](url)
- JIRA: [TICKET-123](url)
- Prototype: [link](url)
- Companion PR: owner/repo#number (if multi-repo)

## Deployment checklist
| Item | Staging | Production |
|------|---------|------------|
| Lever/flag name | What to set | What to set |

## Test plan
- [x] Tests that already pass
- [ ] Manual QA items still to verify
```

For experiment PRs specifically, always include:
- Lever names and their scopes (`:userId`, `:workspaceId`, `:domain`)
- What percentages to configure on staging vs production
- Who owns lever creation if it's not you
