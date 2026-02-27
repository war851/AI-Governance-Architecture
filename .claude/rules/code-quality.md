# Code Quality Rules

## Requirements (MUST)

- Every function must have a single, clear responsibility
- All error paths must be handled explicitly — no empty catch blocks, no swallowed errors
- Use parameterised queries for all database operations — never concatenate user input into queries
- All public APIs must validate input before processing
- Environment-specific values (URLs, keys, ports) must come from environment variables

## Conventions (SHOULD)

- Prefer immutable data structures where the language supports them
- Prefer early returns to reduce nesting depth
- Keep functions under 40 lines; extract helpers when exceeding this
- Group related imports together; separate third-party from local imports
- Use descriptive variable names — avoid single-letter names except in tight loops (`i`, `j`, `k`)

## Anti-Patterns (NEVER)

- NEVER use `any` type in TypeScript — use `unknown` and narrow with type guards
- NEVER suppress linter warnings without a comment explaining why
- NEVER leave `TODO` or `FIXME` comments without an associated issue number
- NEVER use `console.log` for production logging — use a structured logger
- NEVER commit commented-out code — use version control history instead
