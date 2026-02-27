# Git Workflow Rules

## Requirements (MUST)

- Commit messages follow Conventional Commits: `type(scope): description`
  - Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `build`, `ci`
  - Scope: optional, identifies the module/area affected
  - Description: imperative mood, lowercase, no period at end
- Each commit must be atomic — one logical change per commit
- Branch names follow: `feat/short-description`, `fix/short-description`, `chore/short-description`
- All commits must pass linting before being accepted

## Conventions (SHOULD)

- Keep PRs focused: one feature or fix per PR
- PR descriptions must include: summary, motivation, test evidence
- Squash fixup commits before merge to keep history clean
- Tag PR with relevant labels (feature, bugfix, breaking, etc.)

## Anti-Patterns (NEVER)

- NEVER force-push to `main`, `master`, or shared branches
- NEVER commit `.env`, `secrets/`, credentials, or API keys
- NEVER commit generated files (build output, node_modules, .next, dist)
- NEVER rewrite published history — only amend/squash unpushed commits
- NEVER create merge commits on feature branches — rebase instead

## Commit Message Examples

```
feat(auth): add JWT refresh token rotation
fix(api): handle timeout in payment webhook
refactor(db): extract query builder from repository
test(cart): add edge cases for empty cart checkout
docs: update deployment runbook for v2 migration
chore: upgrade dependencies to latest patch versions
```
