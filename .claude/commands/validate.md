# Validation Suite

Run all validation checks and report results.

## Checks

Run each check and record pass/fail. Adapt commands to your project:

1. **Lint**: Run linter(s)
2. **Typecheck**: Run type checker (if applicable)
3. **Tests**: Run test suite
4. **Build**: Run production build

> **Tip**: If your project has a single `npm run validate` or equivalent that runs all checks, use that. Otherwise run each check individually.

## Staged Completeness Check

When run before a commit (i.e., there are staged or unstaged changes):

1. Run `git status --short`
2. For each `??` (untracked) file:
   - Check if any staged source file imports it
   - If found, report as `UNSTAGED DEPENDENCY: <file>`
3. For each ` M` (unstaged modified) file:
   - Check if this file was modified as part of the current work
   - If found, report as `UNSTAGED MODIFICATION: <file>`
4. If no issues: `Staged Completeness: PASS`

This prevents the most common CI failure: files that exist on disk but aren't committed.

## Output

Present results as a summary table:

| Check | Status | Details |
|-------|--------|---------|
| Lint | PASS/FAIL | error count or "clean" |
| Typecheck | PASS/FAIL | error count or "clean" |
| Tests | PASS/FAIL | X passed, Y failed |
| Build | PASS/FAIL | |
| Staged Completeness | PASS/WARN | unstaged dependency count or "clean" |

If any check fails, provide the first few lines of error output for quick diagnosis.
