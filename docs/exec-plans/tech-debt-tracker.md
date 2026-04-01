# Tech Debt & Feature Tracker

Machine-readable task backlog. Single source of truth for all work items.

## Task Format

| ID | Task | Area | Effort | Status | Notes |
|----|------|------|--------|--------|-------|

### ID Prefixes
- `FEAT-#` — New user-facing feature (area = fullstack or any)
- `B-C#` / `B-H#` / `B-M#` / `B-L#` — Backend tasks by priority (Critical/High/Medium/Low)
- `F-C#` / `F-H#` / `F-M#` / `F-L#` — Frontend tasks by priority

### Priority Levels
- **Critical** — Blocking production use
- **High** — High impact, clear need
- **Medium** — Improves quality, not urgent
- **Low** — Nice-to-have polish

### Effort Scale
| Effort | Guideline |
|--------|-----------|
| 0.1d | Config/doc-only changes |
| 0.25d | Single-file fixes |
| 0.5d | New hook/service, moderate refactor |
| 1d | New module, significant feature |
| 2d | Multi-module feature |
| 3d+ | Cross-cutting feature |
| 5d | Major feature |

---

## Critical Priority

| ID | Task | Area | Effort | Status | Notes |
|----|------|------|--------|--------|-------|

## High Priority

| ID | Task | Area | Effort | Status | Notes |
|----|------|------|--------|--------|-------|

## Medium Priority

| ID | Task | Area | Effort | Status | Notes |
|----|------|------|--------|--------|-------|

## Low Priority

| ID | Task | Area | Effort | Status | Notes |
|----|------|------|--------|--------|-------|

## Features

| ID | Task | Area | Effort | Status | Notes |
|----|------|------|--------|--------|-------|
| FEAT-1 | Add remote repo check to setup-workflow command | workflow | 0.25d | done | Phase 0: detect remote, offer to create via gh CLI |
