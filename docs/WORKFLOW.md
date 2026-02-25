# Autonomous Development Workflow

This project uses a self-improving autonomous loop with human steering.

## Workflow Loop

```
User idea → /add-feature → tech-debt-tracker.md → /develop-feature → /validate → /review-pr → PR merge
                                                         ↑                                       |
                                                         |                                       ↓
                                               /audit-service ← ← ← ← ← ← ← ← /retrospective
                                               /doc-garden + /sweep (continuous)
```

## Commands

| Command | Purpose | Cadence |
|---------|---------|---------|
| `/setup-workflow` | First-time project setup | Once |
| `/add-feature` | Interactive backlog intake | On demand |
| `/develop-feature [ID]` | Autonomous end-to-end feature dev | On demand |
| `/validate` | Full CI pipeline locally | Before every PR |
| `/review-pr [PR#]` | Multi-agent self-review | After implementation |
| `/audit-service` | Codebase audit + metrics | Every 5 features |
| `/sweep` | Golden principles enforcement | Every 3 features |
| `/doc-garden` | Documentation freshness scan | With audits |
| `/retrospective` | Workflow self-improvement | Every 10 features |
| `/update-roadmap` | Strategic roadmap refresh | Quarterly (manual) |

## Maintenance Cadence

Maintenance runs are triggered automatically by `/develop-feature` after each merge:

- `/sweep` every **3** features
- `/audit-service` every **5** features
- `/doc-garden` runs with audits
- `/retrospective` every **10** features

State is tracked in `docs/exec-plans/maintenance-cadence.json` (committed to git).

## Parallel Execution Architecture

### Active Work Registry

`docs/exec-plans/active-work.json` (gitignored) tracks running agents:
- Task ID, PID, allocated ports, timestamp
- Prevents concurrent agents from picking the same task
- Auto-cleanup for stale sessions (>24h)

### Port Allocation

When multiple agents run simultaneously, each gets unique ports:
- Slot 0: default ports (e.g., backend 3001, frontend 5173)
- Slot 1: backend +1, frontend +1
- General formula: `base_port + slot`

### Worktree Isolation

Each `/develop-feature` creates an isolated worktree at `.claude/worktrees/<name>/`:
- Eliminates branch/file conflicts between agents
- Phases 0-1 run in main repo (for shared registry)
- Phase 1.5 enters the worktree
- After merge, agent returns to main repo

## Guiding Principles

1. **Humans steer, agents execute** — humans set direction, agents handle implementation
2. **When agents fail, fix the environment** — improve tooling, not agent behavior
3. **Encode taste into tooling** — repeated review findings become lint rules
4. **Corrections cheap, waiting expensive** — fast merge/fix paradigm
5. **Repository is system of record** — all context lives in the repo
6. **Progressive disclosure** — small stable entry points (CLAUDE.md ~60 lines)
7. **Continuous garbage collection** — incremental debt paydown
8. **Parallelism over sequencing** — concurrent agents when possible
