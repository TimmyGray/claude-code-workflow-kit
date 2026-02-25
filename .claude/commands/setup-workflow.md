# Project Workflow Setup

You are an interactive setup agent. Your job is to analyze an existing project, ask clarifying questions, and populate all workflow documentation templates so the autonomous development workflow is fully operational.

## Input
$ARGUMENTS — Optional: short description of the project or specific setup instructions.

## Prerequisites
- The project must be a git repository
- The workflow kit files (CLAUDE.md, docs/, .claude/commands/) must already be copied into the project root
- Node.js project recommended (npm scripts), but adaptable to other stacks

## Workflow

### Phase 1: Project Discovery

Thoroughly analyze the project. Read and understand:

1. **Package manifests**: `package.json` (root, and any sub-packages), `requirements.txt`, `pyproject.toml`, `Cargo.toml`, `go.mod`, etc.
2. **Existing README**: Read any existing `README.md` for project description
3. **Directory structure**: `ls` the root, then key subdirectories. Map the full project layout.
4. **Tech stack detection**:
   - Backend framework (NestJS, Express, FastAPI, Django, Spring, etc.)
   - Frontend framework (React, Vue, Svelte, Angular, etc.)
   - Database (PostgreSQL, MongoDB, MySQL, SQLite, etc.)
   - Testing framework (Vitest, Jest, Pytest, Go test, etc.)
   - Build tools (Vite, Webpack, esbuild, tsc, etc.)
   - Linting (ESLint, Prettier, Ruff, etc.)
   - CI/CD (GitHub Actions, GitLab CI, etc.)
5. **Existing scripts**: Read all npm scripts (or equivalent task runners)
6. **Existing config files**: ESLint config, tsconfig, Docker files, CI workflows
7. **Source code patterns**: Read 3-5 representative source files to understand:
   - Code style and conventions
   - Error handling patterns
   - Logging approach
   - Testing patterns
   - Module/import structure
8. **Existing documentation**: Read any docs/ folder, wiki, or inline docs
9. **Git history**: `git log --oneline -20` to understand recent development patterns
10. **Environment**: Check for `.env.example`, Docker configs, deployment files

### Phase 2: Clarifying Questions

Ask the user clarifying questions in a **single batch** using AskUserQuestion. Only ask questions you couldn't answer from Phase 1. Typical questions:

1. **Project description**: "What does this project do in 1-2 sentences?" (skip if README was clear)
2. **Architecture type**: "Is this a monorepo, monolith, or microservices?" (skip if obvious)
3. **Deployment**: "How is this deployed? (Docker, Vercel, AWS, bare metal, not yet)"
4. **Team size**: "How many developers work on this? (solo, small team 2-5, larger team)"
5. **i18n needs**: "Does this project need internationalization? If so, which languages?"
6. **Current pain points**: "What are the biggest development pain points right now?"
7. **Immediate priorities**: "What are the top 3 things you want to build or fix next?"
8. **Testing status**: "How would you describe current test coverage? (none, minimal, moderate, good)"
9. **Logging**: "What logging library do you use, if any?"
10. **Component library**: "What UI component library do you use?" (frontend only)

### Phase 3: Fill Documentation Templates

Using discovery results and user answers, populate all template files:

#### 3a. CLAUDE.md
- Fill in the quick commands section with actual project commands
- Add project-specific non-negotiable rules (e.g., "All strings use t()" if i18n)
- Update the Quick Reference table if additional docs are created
- Keep it under 60 lines — link to deeper docs

#### 3b. ARCHITECTURE.md
- Write the System Overview section
- Map the Project Structure with actual directory layout
- Document Backend Architecture (modules, data flow, DB schema)
- Document Frontend Architecture (components, state management, API layer)
- Fill in Technology Stack table
- List Environment Variables
- Write Security Considerations

#### 3c. docs/CONVENTIONS.md
- Extract conventions from existing code patterns (Phase 1 step 7)
- Document backend-specific conventions (framework patterns, logging, DB access)
- Document frontend-specific conventions (components, styling, state)
- List mechanically enforced rules (existing ESLint rules, etc.)
- Document environment setup

#### 3d. docs/SECURITY.md
- Document current auth/authz model
- Create threat model based on the application type
- Fill security checklist with project-specific items

#### 3e. docs/RELIABILITY.md
- Document existing error handling patterns
- Document health checks if they exist
- Document logging setup
- Note planned improvements

#### 3f. docs/PRODUCT_SENSE.md
- Write product vision from README + user description
- Create 2-3 user personas based on project type
- Categorize known features into P0/P1/P2
- Add design principles appropriate to the project

#### 3g. docs/PLANS.md
- Create a phased roadmap from user's priorities (Phase 2 Q7)
- Phase 1: Foundation/stability items
- Phase 2: Quality/testing improvements
- Phase 3: Feature development
- Phase 4: Polish/performance

#### 3h. docs/QUALITY_SCORE.md
- Run the actual validation suite and capture initial metrics
- Fill in test counts, lint errors, type errors, build status
- Run `npm audit` or equivalent and capture dependency health
- This becomes the baseline for future sweeps

#### 3i. docs/design-docs/core-beliefs.md
- Keep the generic principles (1-12)
- Add 1-3 project-specific principles based on conventions found
- Remove any that don't apply (e.g., i18n principle if no i18n)

#### 3j. docs/exec-plans/tech-debt-tracker.md
- Populate with initial tasks discovered during analysis:
  - Missing tests for critical paths → add as tasks
  - Security issues found → add as critical/high
  - Code quality issues → add as medium/low
  - User's top 3 priorities (Phase 2 Q7) → add as features
- Assign proper IDs following the prefix convention

#### 3k. docs/DESIGN.md (frontend projects only)
- Document the theme/design system if one exists
- Document component patterns and layout approach
- Note accessibility requirements

### Phase 4: Adapt Commands

Review each command file in `.claude/commands/` and adapt project-specific references:

1. **validate.md**: Update check commands to match project's actual scripts
   - Replace `npm run lint:backend` / `npm run lint:frontend` with actual lint commands
   - Replace test/build commands similarly
   - If monorepo, add per-package checks; if single package, simplify

2. **develop-feature.md**: Update port numbers and server commands
   - Replace `npm run dev:backend` with actual dev server command
   - Replace frontend dev command and ports
   - Update Phase 6 (Playwright validation) browser checks for this project's UI
   - If no separate backend/frontend, simplify the port allocation

3. **review-pr.md**: Update agent review contexts
   - Agent B: update for project's specific quality patterns
   - Agent C: update for project's UI framework and i18n setup (or remove i18n if N/A)
   - Agent D: update test file patterns

4. **sweep.md**: Update violation checklist
   - Adjust code quality checks for the project's stack
   - Update or remove i18n checks if not applicable
   - Update pattern checks for the project's framework

5. **i18n-dev.md**: If i18n is needed, adapt locale list and import patterns. If not needed, note in the file that it's inactive.

6. **add-feature.md**: Update design principle checks to match PRODUCT_SENSE.md

7. **update-roadmap.md**: Verify the command is usable for this project
   - Confirm that `docs/PRODUCT_SENSE.md`, `docs/PLANS.md`, and `docs/exec-plans/tech-debt-tracker.md` exist and are populated
   - The command's web research phase will auto-detect the project's domain from these docs
   - No project-specific edits needed — the command is domain-agnostic by design

### Phase 5: Initial Validation

1. Run the project's full validation suite (or as much as possible)
2. Capture results in QUALITY_SCORE.md
3. Fix any issues discovered during setup (e.g., missing dependencies)
4. Run `git status` to see all changes

### Phase 6: Commit & Report

1. Stage all new/modified documentation files explicitly by name
2. Commit: `"chore: initialize autonomous workflow (setup-workflow)"`
3. Push to main
4. Present a setup summary to the user:

```
## Setup Complete

### Project Profile
- Type: [monorepo/monolith/etc.]
- Stack: [backend] + [frontend] + [database]
- Tests: [X tests found, Y% coverage estimate]

### Documents Created/Updated
- [list of files with brief description of what was filled in]

### Initial Backlog
- [count] tasks added to tech-debt-tracker.md
- [count] critical, [count] high, [count] medium, [count] low, [count] features

### Validation Baseline
- Lint: [status]
- Types: [status]
- Tests: [X passing]
- Build: [status]

### Next Steps
1. Review the generated docs and adjust anything that doesn't match your preferences
2. Run `/add-feature` to add more tasks to the backlog
3. Run `/develop-feature` to start autonomous development
4. Run `/update-roadmap` quarterly to refresh the roadmap with industry research
```

## Important Rules
- Ask questions in a **single batch**, not one at a time
- Never guess about project details — read the code first
- Stage files explicitly — never `git add .` or `git add -A`
- Keep CLAUDE.md under 60 lines — put details in deeper docs
- Adapt commands to the actual project stack — don't leave generic placeholders
- If the project doesn't have tests/lint/CI, note what's missing and add setup tasks to the backlog
- The goal is a fully operational workflow after this command runs — no manual follow-up needed
