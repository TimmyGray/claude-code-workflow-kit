# CLAUDE.md

This file is the table of contents for Claude Code. It points to deeper sources of truth — not a comprehensive manual. (~60 lines, by design.)

## Quick Reference

| What | Where |
|------|-------|
| System architecture | `ARCHITECTURE.md` |
| Code conventions | `docs/CONVENTIONS.md` |
| Workflow & commands | `docs/WORKFLOW.md` |
| Security model | `docs/SECURITY.md` |
| Reliability & errors | `docs/RELIABILITY.md` |
| Product vision | `docs/PRODUCT_SENSE.md` |
| Roadmap | `docs/PLANS.md` |
| Quality metrics | `docs/QUALITY_SCORE.md` |
| Engineering principles | `docs/design-docs/core-beliefs.md` |
| Task backlog | `docs/exec-plans/tech-debt-tracker.md` |
| Maintenance cadence | `docs/exec-plans/maintenance-cadence.json` |

## Commands (Quick)

```bash
# TODO: Fill in after /setup-workflow
npm run dev              # Start development servers
npm run validate         # lint + typecheck + test + build
npm test                 # Run all tests
```

## Claude Code Commands

| Command | When to Use |
|---------|-------------|
| `/setup-workflow` | First-time setup: analyze project, fill docs, create roadmap |
| `/add-feature` | Add feature or task to backlog interactively |
| `/develop-feature` | Pick next task, implement end-to-end, PR, merge |
| `/validate` | Before creating a PR |
| `/review-pr` | After implementation, before merge |
| `/audit-service` | Auto: every 5 features (or manual) |
| `/sweep` | Auto: every 3 features (or manual) |
| `/doc-garden` | Auto: runs with audit (or manual) |
| `/retrospective` | Auto: every 10 features (or manual) |

## Key Rules (Non-Negotiable)

- Stage files explicitly — never `git add .` or `git add -A`
- No `console.log` in production code — use proper logging
- Run `npm run validate` before every PR

<!-- TODO: Add project-specific rules after /setup-workflow -->

For full conventions, see `docs/CONVENTIONS.md`. For workflow details, see `docs/WORKFLOW.md`.
