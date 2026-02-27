# Project: Governed AI
# Stack: TBD — project-agnostic baseline
# Environment: Windows 10 · PowerShell · Cursor + Claude Code

## Architecture Overview

This workspace is configured for dual-agent development:
- **Claude Code** (terminal) handles planning, execution, skills, agents, hooks, and commands
- **Cursor** handles code standards, naming conventions, and architecture patterns
- **Shared layer** owns plans, tasks, logs, and knowledge base

Key directories:
- `.claude/` — Claude Code configuration (settings, rules, commands, agents, skills)
- `.claude/skills/` — All skills: operational (configure-claude-code) + application (VCM, Data Evolution, TDD, Governed Architecture) + artifact templates. Cursor auto-imports these.
- `.cursor/rules/` — Cursor-specific rules (.mdc files)
- `docs/` — Reference material and visualizations
- `src/` — Application source code (structure TBD per project)

Key files:
- `DESIGN.md` — Application design governance index (tracks design artifacts and their status)

## Application Design

For any application design work — value chain mapping, data evolution, TDD golden examples, or governed architecture — read `DESIGN.md` first. It tracks which artifacts exist, their status, and which skills produced them. The skills and artifact templates are in `.claude/skills/`.

## Commands

Commands will be populated once the project stack is chosen. Placeholder:

```bash
# Development
# [start command]     # Dev server

# Testing
# [test command]      # Run test suite
# [test:e2e command]  # End-to-end tests

# Database
# [migrate command]   # Apply migrations
# [reset command]     # Reset local DB (DESTRUCTIVE)

# Quality
# [lint command]      # Lint + format check
# [build command]     # Production build
```

## Coding Standards

- Prefer explicit over implicit in all languages
- All async operations must handle errors explicitly — no silent catches
- Use environment variables for all secrets, URLs, and configuration
- NEVER hardcode credentials, API keys, or connection strings
- Comments only for non-obvious intent, trade-offs, or constraints — never narrate what the code does
- Prefer small, composable functions over large monolithic ones
- Every public function/method needs clear parameter and return type definitions

## Planning Discipline (Boris Pattern)

- ALWAYS start complex tasks in Plan Mode — iterate the plan until solid before executing
- Break work into phases; each phase must have clear completion criteria
- After a mediocre fix: "Knowing everything you know now, scrap this and implement the elegant solution"
- Challenge the AI: "Grill me on these changes; don't proceed until I pass your test"
- Specs before hand-off: reduce ambiguity before auto-mode

## Compounding Engineering

After every correction session, update this CLAUDE.md so the mistake is never repeated.
After every PR merge, review what was learned and encode it here.
This file is a living document — it compounds with every interaction.

This applies across all skills — value chain mapping, TDD, architecture design, or any future skill. Learning, logging, and evolution are handled by the Claude Code environment (this file, `/learn`, rules, git), never by individual skills. One system governs all.

## Git Workflow

- Branch naming: `feat/description`, `fix/description`, `chore/description`
- Commit format: Conventional Commits (`feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`)
- NEVER force-push to `main` or `master`
- NEVER commit secrets, `.env` files, or credentials
- PRs: must have clear description, test evidence, and link to relevant issue

## Known Gotchas

- **Windows environment:** No `osascript` or `notify-send` — use PowerShell for notifications
- **PowerShell syntax:** Use `2>$null` instead of `2>/dev/null`; semicolons or `&&` for chaining
- **Line endings:** Configure git for `core.autocrlf=true` on Windows
- **Path separators:** Use forward slashes in config files; backslashes only in Windows-native commands

## MCP Tools Available

- Cursor browser extension: frontend testing and web automation
- Additional MCP servers: configure per project needs

## Configuration Reference

See @docs/claude-code-configuration-guide.md for the 5-level cascade architecture.
See @docs/claude-code-configuration-sources.md for canonical Anthropic docs and Boris's patterns.
