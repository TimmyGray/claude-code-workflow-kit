# Code Conventions

<!-- Auto-populated by /setup-workflow. Edit as your project evolves. -->

## General

- No `any` types in production code (allowed in tests)
- Max 300 lines per file, max 50 lines per function
- Always handle async errors (try/catch or .catch())
- No `console.log` — use a proper logger or remove

## Backend

<!-- TODO: Backend-specific conventions (logging, validation, error handling, DB access) -->

## Frontend

<!-- TODO: Frontend-specific conventions (component library, styling, state management, i18n) -->

## Git

- Branches: `feat/<kebab>`, `fix/<kebab>`, `chore/<kebab>`
- Commits: descriptive, focused on "why"
- Staging: explicit by name (never `git add .` or `git add -A`)

## Mechanically Enforced Rules

<!-- TODO: List ESLint rules and architecture tests that enforce conventions -->
<!-- Example:
1. No console.log — ESLint `no-console`
2. No cross-module imports — ESLint `no-restricted-imports`
3. Max file/function size — ESLint or architecture test
-->

## Environment

<!-- TODO: .env files, API keys, configuration -->
