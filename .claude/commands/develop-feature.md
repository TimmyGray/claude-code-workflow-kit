# Autonomous Feature Development

You are an autonomous development agent. Your job is to pick a task, implement it end-to-end, create a PR, monitor CI, fix failures, and merge — all without human intervention.

## Input
$ARGUMENTS — Optional task ID from `docs/exec-plans/tech-debt-tracker.md`. If empty, pick the next unstarted highest-priority task.

## Workflow

### Phase 0: Concurrency Setup

Before task selection, set up for safe parallel execution with other agents. **Phases 0-1 must run in the main repo** (before entering a worktree) because `active-work.json` is gitignored and only exists in the main working tree.

> **Sandbox note**: When the agent starts inside a worktree, the sandbox's `.` scope covers only the worktree directory. All writes to `active-work.json` in the main repo (and `git fetch` which writes `FETCH_HEAD`) require `dangerouslyDisableSandbox: true` on Bash calls. This applies to Phases 0, 1 (registry write), 1.5 (git fetch), and 11 (unregister).

1. Read `docs/exec-plans/active-work.json`. If the file is missing or corrupt, back up the corrupt file (`cp active-work.json active-work.json.bak 2>/dev/null || true`), then create a fresh file with: `{"schema_version":1,"sessions":[]}`
2. **Stale cleanup**: For each session entry:
   - If `started_at` is older than 24 hours → remove the entry (safety net for crashed sessions)
   - Otherwise, treat the entry as **active** — do not remove it
3. Write the cleaned registry using atomic rename: write to `active-work.json.tmp`, then `mv active-work.json.tmp active-work.json`
4. **Port allocation**: Find the lowest available slot number (0, 1, 2, ...) not used by any remaining session. Determine base ports from your project config (check `ARCHITECTURE.md` or `.env.example`):
   - Slot 0: default ports (e.g., backend 3001, frontend 5173)
   - Slot N: base_port + N for each service
5. Note the list of currently active `task_id`s — these are off-limits for task selection
6. Store the allocated ports and slot number for use in later phases

### Phase 1: Task Selection
1. Read `docs/exec-plans/tech-debt-tracker.md`
2. If task ID provided, find that task. Otherwise, select the next task with status `todo` and highest priority (critical > high > medium > low)
3. **Skip active tasks**: When selecting the next `todo` task, skip any task whose ID appears in the active-work registry (from Phase 0 step 5). If a specific task ID was provided via `$ARGUMENTS` and it's already active in the registry, **warn and stop**
4. Display the selected task and confirm understanding
5. **Register in active-work registry**: Add a session entry to `docs/exec-plans/active-work.json` with:
   - `task_id`: the selected task ID
   - `branch`: the branch name to be created (e.g., `feat/<kebab-case-task-name>`)
   - `started_at`: current ISO 8601 timestamp
   - `ports`: `{"backend": <allocated_backend_port>, "frontend": <allocated_frontend_port>}`
6. Write the updated registry using atomic rename (write to `.tmp`, then `mv`)
7. **Post-write verification**: Re-read `active-work.json` and confirm that:
   - Own entry is present
   - No other entry has the same slot or task_id
   - If a conflict is detected, pick a new slot/task and retry from step 1 (max 3 retries)

### Phase 1.5: Worktree Isolation

Isolate the working tree so multiple `/develop-feature` agents can run concurrently without branch conflicts.

1. **Store main repo root** for later reference: Run `git rev-parse --show-toplevel` and remember this path
2. **Enter an isolated worktree** using the `EnterWorktree` tool with `name: "feat-<task-kebab>"`. This creates a fresh copy of the repo at `.claude/worktrees/<name>/` with its own branch.
   - If `EnterWorktree` fails, skip — the agent is already isolated
3. **Verify isolation**: Run `git rev-parse --show-toplevel` and confirm the path contains `.claude/worktrees/`
4. **Install dependencies**: Run dependency install commands appropriate for the project (e.g., `npm install`, `pip install -r requirements.txt`, etc.)
5. **Sync remote refs**: Run `git fetch origin main` to ensure `origin/main` is up-to-date. **Never run `git checkout main`** inside a worktree.

### Phase 2: Context Gathering
1. Read `ARCHITECTURE.md` for system overview
2. Based on the task area:
   - Backend work → read relevant backend source files
   - Frontend work → read frontend docs and relevant components
   - Full-stack → read all of the above
3. Read `docs/design-docs/core-beliefs.md` for engineering principles
4. Read `docs/CONVENTIONS.md` for code standards

### Phase 3: Branch & Plan
1. Create a feature branch: `git checkout -b feat/<kebab-case-task-name> origin/main`
2. **Execution Plan (required for tasks > 1 day effort)**:
   - Create `docs/exec-plans/active/<feature>.md` with:
     - Task description and acceptance criteria
     - Implementation phases with estimated effort
     - Files to create/modify
     - Decision log (fill in as you make choices)
     - Risks and open questions
3. Plan the implementation: list files to create/modify, tests to write

### Phase 4: Implementation
1. Implement the changes following project conventions (see `docs/CONVENTIONS.md`)
2. Write tests for new functionality
3. Follow patterns from `docs/references/` if applicable
4. **Update execution plan** after each significant decision

### Phase 5: Validation + Staged-File Completeness
1. Run validation checks — fix failures, max 3 retries per check:
   ```
   npm run validate
   ```
   Or run individual checks as defined in your project's validate command.
2. After all checks pass, run **staged-file completeness check**:
   - Run `git status --short`
   - For each `??` (untracked) file, check if any file you modified/created imports it
   - For each ` M` (unstaged modified) file, check if it was changed during implementation
   - If an untracked/unstaged file is imported by any changed file, stage it immediately
   - After staging any new files, re-run typecheck and build to confirm clean state
3. This step prevents CI failures caused by files that exist on disk but aren't committed

### Phase 6: Deep Validation (Port-Aware)

Use the ports allocated in Phase 0. This ensures multiple agents can run dev servers simultaneously.

1. Start dev servers in background using allocated ports (check ARCHITECTURE.md for the correct commands)
2. Wait for servers to be ready (poll health endpoints, max 30s)
3. **Deep browser validation** (if project has a frontend):
   - Use `browser_navigate` to the frontend URL
   - Use `browser_snapshot` to capture accessibility tree
   - Navigate the specific user flow affected by this feature
   - Use `browser_console_messages(level: "error")` — zero JS errors required
   - Use `browser_network_requests` — verify API routes return expected status codes
   - Use `browser_take_screenshot` to capture evidence for the PR
4. **API validation** (if backend-only): test endpoints with curl or similar
5. **Check logs for errors**: Read server output, flag any ERROR level entries
6. Kill dev servers precisely using port-specific commands:
   - `lsof -ti:<port> | xargs kill 2>/dev/null || true`
7. If validation reveals critical issues, fix and return to Phase 5 (max 3 retry cycles)

### Phase 7: Commit & PR
1. Run `git status --short` one final time
2. **Working directory assertion**: Confirm you're at the repo root
3. Stage each file explicitly by name (never `git add .` or `git add -A`)
4. Cross-check: `git diff --cached --name-only` includes all files created/modified
5. Commit with a descriptive message focused on "why"
6. Push: `git push -u origin feat/<branch-name>`
7. Create PR via `gh pr create` with summary and test plan

### Phase 8: CI Monitor & Fix Loop
1. Wait for checks: `sleep 10`, then `gh pr checks <PR#> --watch --interval 30`
2. **In parallel**: launch a `code-review-advisor` subagent for speculative self-review
3. If CI passes → proceed to Phase 9
4. If CI fails:
   - Read failure logs: `gh run view <RUN_ID> --log-failed`
   - Analyze and fix the code
   - Stage, commit, push the fix
   - Return to step 1 (max 3 CI fix cycles)
5. If CI still fails after 3 cycles, stop and report the blocker

### Phase 9: Self-Review & Fix Loop
1. Use the speculative review from Phase 8, or run `/review-pr` now
2. The review produces: `VERDICT: APPROVE` or `VERDICT: REQUEST_CHANGES`
3. If `VERDICT: APPROVE` → proceed to Phase 10
4. If `VERDICT: REQUEST_CHANGES`:
   - Fix all blocking issues
   - Stage, commit, push fixes
   - Return to Phase 8 step 1 (max 2 review-fix cycles)
5. If still REQUEST_CHANGES after 2 cycles, stop and report remaining issues

### Phase 9b: Root Cause Analysis & Immediate Learning

**Activation**: Only when Phase 9 had `REQUEST_CHANGES` (skip if APPROVE on first pass).

1. **Collect findings**: Gather all critical/warning findings from reviews
2. **Classify each by root cause category**:
   - `CONV` — Convention gap
   - `OVER` — Overgeneralization
   - `RAIL` — Missing guard rail (no lint rule/test)
   - `PROC` — Workflow/process bug
   - `MODEL` — Incomplete mental model
   - `COPY` — Copy-paste drift
3. **Write structured entries** to `pitfalls.md` in the Claude Code auto-memory directory
4. **Check promotion threshold**: If 2+ occurrences of the same category, promote:
   - `CONV` → Update `docs/CONVENTIONS.md` + `review-pr.md` agent prompt
   - `RAIL` → Add lint rule or architecture test
   - `PROC` → Add safeguard to `develop-feature.md`
5. **Append root cause summary** to the PR description

### Phase 10: Auto-Merge (Conflict-Aware)
1. Verify PR is mergeable: `gh pr view --json mergeable,mergeStateStatus`
2. If not mergeable:
   - Attempt rebase: `git fetch origin main && git rebase origin/main`
   - If rebase succeeds, force-push: `git push --force-with-lease`
   - If conflicts in **bookkeeping files only**: resolve by accepting main + re-applying changes
   - If conflicts in **source code**: abort rebase, stop and report
   - Retry the merge check. If still not mergeable after one rebase attempt, stop and report
3. Merge: `gh pr merge --squash --delete-branch`
4. If merge fails with **"head branch is not up to date"**:
   - `git fetch origin main && git rebase origin/main`
   - Resolve any conflicts (bookkeeping only — abort and report if source code conflicts)
   - Run the project's validation suite to confirm nothing broke
   - `git push --force-with-lease`
   - Wait for CI checks to pass on the updated branch
   - Retry: `gh pr merge --squash --delete-branch`
   - If merge still fails after one recovery attempt, stop and report
5. Switch to main repo root (from Phase 1.5), run `git pull origin main`

### Phase 11: Bookkeeping (Post-Merge)

> **Important**: All Phase 11-12 operations run from the **main repo working tree**.

1. Update task status in `docs/exec-plans/tech-debt-tracker.md` to `done`
2. If an exec plan was created, move it to `docs/exec-plans/completed/`
3. Stage bookkeeping files explicitly, commit, push
4. **Unregister from active-work registry**: Remove own entry from `active-work.json`

### Phase 12: Maintenance Cadence Check

1. Read `docs/exec-plans/maintenance-cadence.json`
   - If missing, create with default thresholds (sweep: 3, audit: 5, retrospective: 10)
2. **Recovery check**: If `pending_runs` is non-empty, re-launch them now
3. Increment all three counters by 1
4. Determine which maintenance tasks are due (counter >= threshold)
5. Reset due counters to 0, update `last_runs` dates
6. Write `pending_runs`, commit and push
7. **Launch due maintenance as parallel background Task subagents**:
   - **Sweep**: "Read `.claude/commands/sweep.md` and execute all instructions."
   - **Audit + Doc-garden**: "Read `.claude/commands/audit-service.md` and execute. Then read `.claude/commands/doc-garden.md` and execute."
   - **Retrospective**: "Read `.claude/commands/retrospective.md` and execute all instructions."
8. Clear `pending_runs`, commit and push
9. If no maintenance due, report the countdown

## Important Rules
- Never skip validation. Fix errors before creating the PR.
- Stage files explicitly — never use `git add .` or `git add -A`
- If a task touches > 15 files, break it into subtasks first
- If tests fail after 3 attempts, stop and report the issue
- Always kill dev servers using port-specific `lsof` commands
- The staged-file completeness check is critical — it prevents the #1 cause of CI failures
- Max retry limits are hard limits: do not exceed them
- Execution plans are living documents: update as you make decisions
