# Security Model

<!-- Auto-populated by /setup-workflow -->

## Current Security Model

<!-- TODO: Describe authentication, authorization, input validation, etc. -->

## Threat Model

<!-- TODO: List identified threats with severity and mitigation status -->
<!-- Example:
| Threat | Severity | Mitigation |
|--------|----------|------------|
| API key exposure | Critical | Env vars only, .gitignore |
| XSS in user content | High | Sanitization library |
-->

## Security Checklist (for Code Reviews)

- [ ] No hardcoded secrets, API keys, or credentials
- [ ] All user input is validated at the boundary (DTOs, schemas)
- [ ] Output is sanitized before rendering
- [ ] File paths are validated (no path traversal)
- [ ] Rate limiting on sensitive endpoints
- [ ] CORS properly configured
- [ ] Error messages don't leak internal details
- [ ] Dependencies checked for known vulnerabilities
