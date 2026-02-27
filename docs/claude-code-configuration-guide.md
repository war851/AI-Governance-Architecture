# Claude Code: Complete Configuration Architecture Guide
> **Sources:** Anthropic Official Docs · howborisusesclaudecode.com · Anthropic Engineering Best Practices  
> **Version:** February 2026 · Project/Infrastructure Agnostic

---

## Quick Reference: The 5-Level Cascade

```
PRECEDENCE (highest → lowest when conflicting)
┌─────────────────────────────────────────────────────────────────┐
│ 1. MANAGED POLICY  /etc/claude-code/CLAUDE.md                  │ ← org-wide, IT-deployed
│ 2. USER MEMORY     ~/.claude/CLAUDE.md                          │ ← personal, all projects
│ 3. PROJECT MEMORY  ./CLAUDE.md  OR  ./.claude/CLAUDE.md         │ ← team, git-committed
│ 4. PROJECT RULES   ./.claude/rules/*.md                         │ ← modular, path-scoped
│ 5. LOCAL OVERRIDE  ./CLAUDE.local.md                            │ ← personal, gitignored
│ ─────────────────────────────────────────────────────────────── │
│ + AUTO MEMORY      ~/.claude/projects/<proj>/memory/MEMORY.md   │ ← Claude self-written
└─────────────────────────────────────────────────────────────────┘

LOADING BEHAVIOUR
• Levels 1–3 + 5: Loaded IN FULL at session start
• Level 4 rules: Loaded at start (path-scoped rules only when matching files are touched)
• Auto memory: First 200 lines of MEMORY.md injected at start; topic files on-demand
• Parent-dir CLAUDE.md files: Recursively loaded upward (not including /)
• Sub-dir CLAUDE.md files: Loaded on demand when Claude reads files in that subtree
```

---

## Level 1 — Managed Policy (Enterprise / DevOps)

### What It Is
A system-level CLAUDE.md deployed by IT or DevOps to every developer machine. It cannot be overridden by any user or project setting. Think of it as your organisation's constitution for AI behaviour.

### File Location
| OS | Path |
|----|------|
| macOS | `/Library/Application Support/ClaudeCode/CLAUDE.md` |
| Linux / WSL | `/etc/claude-code/CLAUDE.md` |
| Windows | `C:\Program Files\ClaudeCode\CLAUDE.md` |

> Also supports `managed-settings.json` in the same directory for JSON-based policies.

### Purpose
- Security policies that must be enforced organisation-wide
- Compliance requirements (GDPR, SOC 2, HIPAA)
- Standardised tooling references shared across all teams
- Forbidden actions (e.g., never push to `main` directly)

### ✅ Positive Examples

```markdown
# [ACME Corp] Managed AI Policy

## Security Constraints
- NEVER read or modify files matching `**/.env*`, `**/secrets/**`, `**/credentials/**`
- NEVER execute `curl` or `wget` to external endpoints without explicit user confirmation
- NEVER commit directly to `main` or `master` branches
- NEVER store or log API keys, tokens, or passwords in any output

## Compliance
- All generated code must include SPDX licence headers: `// SPDX-License-Identifier: MIT`
- PII fields must always use the approved anonymisation helper: `utils/pii.ts`

## Required Tooling
- Use `pnpm` (not npm or yarn) for all Node.js package operations
- Use the internal proxy `https://registry.acme.internal` for package installs

## Escalation
- For architectural decisions affecting >3 services, open a design doc before coding
```

### ❌ Negative Examples (What NOT to Do)

```markdown
# BAD: Too vague — Claude can't act on "be careful"
- Be careful with sensitive data

# BAD: Contradicts itself
- Always run tests before committing
- Never run tests in the managed environment  ← conflict

# BAD: Over-specified implementation detail that will break
- Always use ESLint version 8.2.1 exactly  ← pin in package.json, not here

# BAD: Blank or near-empty managed policy
# (wastes the only non-overridable layer)
```

### Limits & Caveats
- Requires administrator/root privileges to write
- Deploy via MDM (Jamf, Intune), Ansible, or Chef — not manually
- Changes require re-deployment; no live reload
- Cannot override this level from any project or user file
- `allowManagedHooksOnly: true` in `managed-settings.json` blocks all user/project hooks

### Template

```markdown
# [ORG NAME] — Managed Claude Code Policy
# Deployed by: DevOps/IT  |  Last updated: YYYY-MM-DD

## 🔒 Security — Non-Negotiable
- NEVER read: `.env*`, `secrets/`, `credentials/`, `*.pem`, `*.key`
- NEVER execute network calls to external IPs without user confirmation
- NEVER commit secrets or tokens in any form

## ⚖️ Compliance
- [Add company-specific compliance rules here]

## 🛠 Approved Tooling
- Package manager: [pnpm / npm / yarn]
- Registry: [internal URL or public]
- CI/CD: [tool name + relevant commands]

## 🚦 Git Policy
- Branch strategy: [GitFlow / trunk-based / etc.]
- Protected branches: [main, release/*, etc.]
- Commit convention: [Conventional Commits / custom]

## 📞 Escalation
- [When to stop and ask a human]
```

### Boris Best Practice
> Use `managed-settings.json` alongside this file to enforce `allowManagedHooksOnly: true` in regulated environments. Check settings into a config-management repo and auto-deploy with your existing MDM pipeline.

---

## Level 2 — User Memory (Personal Global)

### What It Is
Your personal CLAUDE.md that applies to **every project** on your machine. This is where you encode your individual coding style, preferred tools, and workflow habits — things you'd never want to repeat in every project.

### File Location
```
~/.claude/CLAUDE.md
```

### Purpose
- Personal coding style preferences (naming, indentation, comment style)
- Preferred CLI tools and shortcuts
- Communication style (terse vs. explanatory responses)
- Personal workflow patterns (how you like PRs structured, commit messages written)
- Global shortcuts and aliases

### ✅ Positive Examples

```markdown
# My Personal Claude Preferences

## Communication Style
- Be concise. Skip preamble ("Sure!", "Of course!") — go straight to the answer
- When I ask for code, give me the code first, explanation after if needed
- Use inline comments only for non-obvious logic; never comment the obvious
- If you are uncertain, say so explicitly rather than guessing

## Code Style (Personal Defaults — Projects Override These)
- Prefer functional style over OOP where both are equally clear
- TypeScript: always use `type` over `interface` unless extending
- Python: use f-strings, not `.format()` or `%`
- Commit messages: Conventional Commits format (`feat:`, `fix:`, `chore:`, etc.)

## My Tools
- Shell: zsh with oh-my-zsh; use `z` for directory jumping
- Package manager: pnpm for Node.js projects
- Run `bun` for scripts where startup speed matters
- Editor: Cursor (treat it like VS Code for all purposes)
- Docker: always use `docker compose` (v2), not `docker-compose` (v1)

## Workflow
- Before implementing anything complex, ask me to confirm the plan
- When fixing bugs, show me the root cause before showing the fix
- When I say "clean this up", focus on readability, not micro-optimisations
```

### ❌ Negative Examples

```markdown
# BAD: Project-specific things don't belong here
- The API base URL is https://api.myproject.com/v2  ← goes in CLAUDE.local.md

# BAD: Overly restrictive for a personal file
- Never use async/await  ← you'll override this in every single project

# BAD: Contradicts itself across sections
- Be verbose and explain everything
- ...
- Keep responses short  ← pick one

# BAD: Copy-pasting boilerplate that doesn't reflect YOU
- [50 lines of generic "best practices" from a blog post]
```

### Limits & Caveats
- Not shared with team — only on your machine
- If you work across multiple machines, you must sync this manually (dotfiles repo)
- Too much content here means project-level rules get diluted
- Applies to ALL projects: be careful about project-specific commands that could conflict

### Template

```markdown
# Personal Claude Configuration
# ~/.claude/CLAUDE.md  |  Machine: [hostname]

## 🗣 Communication
- Response style: [concise/detailed/bullet-heavy]
- [List your pet peeves from AI responses]

## 💅 Code Style Defaults
- Language preferences: [TS/JS/Python/etc specifics]
- Naming conventions: [camelCase, snake_case, etc.]
- Comment philosophy: [when to comment, when not to]

## 🔧 My Tools & Environment
- Shell: [zsh/bash/fish] with [plugins]
- Package managers: [per ecosystem]
- Key CLIs I use: [list with one-line purpose]
- Docker setup: [compose version, default flags]

## 🔄 My Workflow
- Planning preference: [always plan first / dive in / depends]
- PR style: [atomic commits / squash / etc.]
- Testing approach: [TDD / test after / etc.]

## 📝 Commit Convention
- Format: [template]
- Example: [example]
```

### Boris Best Practice
> Boris keeps his personal global file minimal and prefers investing in project-level `CLAUDE.md` files that are team-shared. After any correction session, end with: **"Update CLAUDE.md so you don't make that mistake again."** Claude is highly effective at writing its own rules.

---

## Level 3 — Project Memory (Team-Shared)

### What It Is
The **primary** configuration file for a project. Checked into git and shared with the whole team. This is the file Boris's team maintains together — multiple commits per week, updated via PR review. This is where you build "Compounding Engineering."

### File Location
```
./CLAUDE.md          ← preferred for simple projects
./.claude/CLAUDE.md  ← preferred for complex projects (keeps root clean)
```

> Use `@import` syntax to pull in sub-files and keep it modular:
> ```markdown
> See @README for project overview and @package.json for available commands.
> # Additional Instructions
> - git workflow: @docs/git-instructions.md
> ```

### Purpose
- Project architecture overview + key directory structure
- Team coding standards and conventions
- Common commands (build, test, deploy, migrate)
- MCP tools the team uses
- PR and commit conventions
- Known gotchas and "don't do this" patterns
- Links to internal docs and runbooks

### ✅ Positive Examples

```markdown
# Project: [Name]
# Stack: [e.g., Next.js 14 + Supabase + n8n]

## Architecture Overview
This is a [brief description]. The main components are:
- `src/app/` — Next.js App Router pages and API routes
- `src/lib/` — Shared utilities and service clients
- `supabase/` — Database migrations, edge functions, seed data
- `n8n/` — Workflow exports (JSON); import via n8n UI
- `docs/` — Architecture decisions and runbooks

## Commands
```bash
pnpm dev          # Start dev server (http://localhost:3000)
pnpm test         # Run Vitest test suite
pnpm test:e2e     # Run Playwright end-to-end tests
pnpm db:migrate   # Run pending Supabase migrations
pnpm db:reset     # Reset local DB to seed state (DESTRUCTIVE)
pnpm lint         # ESLint + Prettier check
pnpm build        # Production build
```

## Coding Standards
- TypeScript strict mode is enabled — never use `any`
- All async functions must handle errors explicitly (no silent catches)
- Database queries: always use parameterised queries via Supabase client
- File naming: `kebab-case.ts` for modules, `PascalCase.tsx` for components
- State management: Zustand (not Redux, not Context for global state)

## Supabase Conventions
- Migrations: named `YYYYMMDDHHMMSS_description.sql`
- RLS: every new table MUST have RLS enabled; no exceptions
- Edge functions: always validate JWT before executing business logic

## Git Workflow
- Branch: `feat/description`, `fix/description`, `chore/description`
- Commits: Conventional Commits (`feat:`, `fix:`, `refactor:`, `test:`, `docs:`)
- PRs: Must pass all CI checks; at least 1 human review + tag `@.claude` for AI review
- NEVER force-push to `main` or `develop`

## Known Gotchas
- The `useRouter()` hook from Next.js 14 requires `"use client"` — server components must use `redirect()`
- Supabase `realtime` channels must be unsubscribed in cleanup functions or you'll leak connections
- n8n webhook URLs change on re-import; always use the environment variable `N8N_WEBHOOK_BASE`
```

### ❌ Negative Examples

```markdown
# BAD: No structure — Claude can't parse a wall of text
This project is a web application that uses React and has a database and an API layer and 
we usually do things with TypeScript and sometimes we test things...

# BAD: Stale information left uncorrected
- Use yarn (team switched to pnpm 6 months ago, but nobody updated this)

# BAD: Including secrets or environment-specific values
- DATABASE_URL=postgresql://user:password@localhost:5432/mydb  ← NEVER

# BAD: Generic filler without project-specific content
- Write good code
- Test your changes
- Follow best practices
```

### Limits & Caveats
- Entire file is loaded at every session start — keep it focused; quality > quantity
- The file becomes part of Claude's context window; bloated files waste tokens
- Use `@imports` to modularise — don't make one file with 500 lines
- Max `@import` depth is 5 hops
- Auto-generated by `/init` — always review and curate the output
- Path-scoped rules belong in Level 4 (`.claude/rules/`), not here

### Template

```markdown
# Project: [PROJECT NAME]
# Stack: [Tech stack summary]
# Docs: [Link to internal wiki / README]

## 🗺 Architecture Overview
[2–4 sentence description of what this project is and does]

Key directories:
- `[dir]/` — [purpose]
- `[dir]/` — [purpose]
- `[dir]/` — [purpose]

## ⚡ Commands
```bash
# Development
[start command]     # [what it does + URL]
[test command]      # [what it does]

# Database
[migrate command]   # [what it does]
[reset command]     # [what it does — mark DESTRUCTIVE if relevant]

# CI/CD
[lint command]      # [what it does]
[build command]     # [what it does]
```

## 📐 Coding Standards
- [Language-specific rules — be specific, not generic]
- [Naming conventions]
- [Error handling approach]
- [Testing requirements]

## 🗄 Data Layer Conventions
- [ORM/query builder rules]
- [Migration naming]
- [Security requirements (RLS, parameterised queries, etc.)]

## 🔀 Git Workflow
- Branch naming: [pattern]
- Commit format: [format + example]
- PR process: [steps]
- Protected branches: [list]

## ⚠️ Known Gotchas
- [Actual project-specific traps and non-obvious behaviours]

## 🧰 MCP Tools Available
- [tool name]: [when and how to use it]
```

### Boris Best Practice
> Tag `@.claude` on PRs to add learnings to `CLAUDE.md` as part of the PR itself. Boris calls this "Compounding Engineering" — the project gets smarter with every merge. Also: after every correction, say **"Update CLAUDE.md so you don't make that mistake again."**

---

## Level 4 — Project Rules (Modular & Path-Scoped)

### What It Is
Modular rule files inside `.claude/rules/` that extend the main project `CLAUDE.md`. Each file focuses on a single topic (testing, API design, security). Rules can be **path-scoped** using YAML frontmatter — they only activate when Claude is working with matching files.

### File Location
```
.claude/rules/
├── code-style.md          ← applies to all files
├── testing.md             ← applies to all files
├── security.md            ← applies to all files
├── frontend/
│   ├── react.md           ← scoped: src/**/*.tsx
│   └── styles.md          ← scoped: src/**/*.css
└── backend/
    ├── api.md             ← scoped: src/api/**
    └── database.md        ← scoped: supabase/**
```

> All `.md` files in `.claude/rules/` (recursive) are automatically loaded as project memory. Path-scoped rules activate only when Claude works with matching files.

### Purpose
- Keep `CLAUDE.md` lean by extracting domain-specific rules
- Apply context-sensitive rules (React rules only when editing `.tsx` files)
- Allow teams to own their domain's rules independently
- Enforce API contracts, test conventions, security requirements at the file level
- Symlink shared rules across multiple repos: `ln -s ~/shared-claude-rules .claude/rules/shared`

### ✅ Positive Examples

#### `testing.md`
```markdown
---
paths:
  - "**/*.test.ts"
  - "**/*.spec.ts"
  - "tests/**/*"
---
# Testing Rules

- Use Vitest, not Jest — the APIs are similar but import from `vitest`
- Test naming: `describe('[ComponentName]', () => { it('[action] [expected result]') })`
- Always test: happy path, error path, and edge cases
- Mock external dependencies at the module level, not inline
- Never use `setTimeout` in tests — use `vi.useFakeTimers()` instead
- Each test must be independent; never share mutable state between tests
- Snapshot tests are forbidden — they become stale and hide real failures
```

#### `api.md`
```markdown
---
paths:
  - "src/api/**/*.ts"
  - "src/app/api/**/*.ts"
---
# API Development Rules

- Every endpoint MUST validate input with Zod before processing
- Use the standard error response format:
  ```json
  { "error": { "code": "VALIDATION_ERROR", "message": "...", "details": [...] } }
  ```
- All endpoints must include OpenAPI JSDoc comments
- Authentication: validate JWT using `lib/auth/validate.ts` — never roll your own
- Rate limiting: all public endpoints must use the `rateLimit` middleware
- Never return raw database errors to clients — map to safe error messages
```

#### `database.md`
```markdown
---
paths:
  - "supabase/**"
  - "src/lib/db/**"
---
# Database Rules

- NEVER use `supabase.from('table').select('*')` in production code — always specify columns
- Row Level Security (RLS): every new table needs a policy before the migration is merged
- Migration file format: `YYYYMMDDHHMMSS_verb_noun.sql` (e.g., `20260215143000_add_user_preferences.sql`)
- Use `supabase.rpc()` for complex queries — keeps business logic in the DB layer
- Foreign keys: always explicit `ON DELETE` behaviour — no implicit cascades
```

### ❌ Negative Examples

```markdown
# BAD: Path-scoped rule with overly broad glob that fires for everything
---
paths:
  - "**/*"
---
# This defeats the purpose of path-scoping — just put it in CLAUDE.md

---

# BAD: Contradicts the main CLAUDE.md
# (CLAUDE.md says use Vitest, rules/testing.md says use Jest)

# BAD: Rules without examples when the pattern is non-obvious
- Always use the correct error format  ← what format? show it!

# BAD: Circular symlinks in .claude/rules/ (Claude detects and skips, but still confusing)
```

### Limits & Caveats
- Path-scoped rules use glob patterns — test globs carefully (typos silently drop rules)
- All rules in `.claude/rules/` ARE loaded unconditionally if they have no `paths:` frontmatter
- Supports brace expansion: `src/**/*.{ts,tsx}` and multi-pattern arrays
- Symlinks are followed — useful for sharing across repos but creates implicit dependencies
- User-level rules at `~/.claude/rules/` are loaded BEFORE project rules; project rules take precedence

### Template

#### Generic Rule File (no path scope)
```markdown
# [Topic] Rules
# .claude/rules/[topic].md

## Overview
[One sentence on what this rule file governs]

## Requirements (MUST)
- [Mandatory behaviour — use strong language]
- [Another mandatory rule]

## Conventions (SHOULD)
- [Preferred patterns]
- [Style guidance]

## Anti-Patterns (NEVER)
- NEVER [dangerous or incorrect pattern] because [reason]
- NEVER [another anti-pattern]

## Examples
### ✅ Correct
```[language]
[good example]
```
### ❌ Incorrect
```[language]
[bad example and why]
```
```

#### Path-Scoped Rule File
```markdown
---
paths:
  - "[glob pattern matching the relevant files]"
  - "[second pattern if needed]"
---
# [Domain] Rules — Active for: [human description of scope]

[Content as above]
```

### Boris Best Practice
> Create a `~/.claude/rules/` directory with your personal rules that apply across all projects (your universal anti-patterns). Then use `.claude/rules/` for team/project rules. Use symlinks to share a `shared-security.md` across multiple repos without duplication.

---

## Level 5 — Local Override (Personal, Project-Specific)

### What It Is
A `CLAUDE.local.md` file in your project root — **automatically gitignored** by Claude Code. This is your personal sandbox for project-specific preferences that the team doesn't need: your local URLs, your test data, your personal debugging notes.

### File Location
```
./CLAUDE.local.md    ← auto-added to .gitignore
```

### Purpose
- Your local dev environment specifics (ports, sandbox URLs, test DB credentials reference)
- Personal shortcuts for a specific project
- WIP notes that guide Claude during an active investigation
- Machine-specific overrides that won't work for others
- Experimental rules you're testing before promoting to `CLAUDE.md`

### ✅ Positive Examples

```markdown
# Local Overrides for [Project Name]
# NOT committed to git — personal to this machine

## My Local Environment
- Local dev server: http://localhost:3000
- Local Supabase: http://localhost:54321 (Studio: http://localhost:54323)
- n8n local instance: http://localhost:5678
- Test user credentials: see `.env.local` (use the `TEST_USER_EMAIL` var)

## My Preferred Test Data
- Use customer ID `cus_test_groot_001` for all payment flow testing
- Seed script for my test data: `pnpm db:seed:groot` (not in package.json, it's my local alias)

## Active Investigation Notes (remove when done)
- Bug #482: The race condition only reproduces when the webhook fires within 200ms of the session
  creation. Suspect `supabase/functions/process-payment/index.ts` lines 45-67.
- Temp: disable the rate limiter locally: comment out line 12 in `middleware.ts`

## Personal Shortcuts
- When I say "deploy to staging", run: `git push origin HEAD && gh pr create --draft`
```

### ❌ Negative Examples

```markdown
# BAD: Actual secrets (even though gitignored, this is risky)
DATABASE_PASSWORD=supersecretpassword123  ← use .env.local for this, not CLAUDE.local.md

# BAD: Rules that should be in CLAUDE.md (team would benefit from knowing this)
- The Supabase realtime subscription must be cleaned up in useEffect  ← promote this to CLAUDE.md

# BAD: Keeping stale investigation notes forever
- (Notes from a bug fixed 3 months ago that now confuse Claude about current state)

# BAD: Using this to circumvent team standards
- Ignore the TypeScript strict mode requirement  ← this is sabotage, not a personal preference
```

### Limits & Caveats
- Auto-gitignored — you can edit it freely but it will never be shared
- If you use multiple git worktrees, `CLAUDE.local.md` only exists in one; use `~/.claude/CLAUDE.md` imports for cross-worktree personal instructions
- Use `@~/.claude/my-project-instructions.md` in this file to pull in shared personal preferences
- This is the right place to iterate on rules before promoting them to `CLAUDE.md`

### Template

```markdown
# [Project Name] — Local Overrides
# ./CLAUDE.local.md  |  Auto-gitignored  |  Machine: [hostname]

## 🖥 My Local Environment
- Dev server: [URL]
- Database (local): [connection info reference, NOT the actual credentials]
- [Other local services and their URLs]

## 🧪 My Test Data
- [Preferred test fixture IDs]
- [Local seed scripts]
- [Test account references]

## 🔬 Active Investigation (clean up when done)
- [Current bug context]
- [Temporary workarounds — label with date so you remember to remove them]

## ⌨️ Personal Shortcuts (this project only)
- When I say "[shorthand]", do: [full instruction]

## 🧫 Experimental Rules (candidates for CLAUDE.md)
- [Rule I'm testing — if it works for a week, promote it]
```

### Boris Best Practice
> One engineer on Boris's team maintains a **notes directory for every task/project**, updated after every PR, and points `CLAUDE.md` at it. `CLAUDE.local.md` is the right place to start that directory reference before it becomes a team standard.

---

## Beyond CLAUDE.md: The Full Configuration Stack

The 5 CLAUDE.md levels handle **memory** (what Claude knows). The following handle **behaviour** (how Claude acts). Together they form the complete setup.

### settings.json — The Behaviour Layer

Three scopes, same precedence model as CLAUDE.md:

```
~/.claude/settings.json          ← User: personal global
.claude/settings.json            ← Project: team-shared (commit to git)
.claude/settings.local.json      ← Local: personal overrides (gitignored)
```

**Starter project `settings.json`:**
```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(pnpm run *)",
      "Bash(pnpm test *)",
      "Bash(git *)",
      "Bash(docker compose *)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./.aws/**)",
      "Bash(curl *)",
      "Bash(rm -rf *)"
    ],
    "defaultMode": "acceptEdits"
  },
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "0"
  }
}
```

> **Boris's tip:** Use `/permissions` to pre-allow common safe commands. Never use `--dangerously-skip-permissions` in a shared environment — use the `sandbox` mode instead for unattended runs.

### Hooks — Deterministic Automation

Hooks are shell commands Claude runs automatically at lifecycle events. They are the **deterministic** "must-do" layer that complements the "should-do" suggestions in `CLAUDE.md`.

**Hook events:**
| Event | Fires When | Common Use |
|-------|-----------|------------|
| `PreToolUse` | Before Claude uses any tool | Block dangerous commands, validate inputs |
| `PostToolUse` | After Claude uses any tool | Auto-format, run linter, log actions |
| `Stop` | Claude finishes a response | Desktop notifications, CI triggers |
| `SessionStart` | Session begins | Inject dynamic context (date, branch, env) |

**Starter hook config in `.claude/settings.json`:**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{
          "type": "command",
          "command": "pnpm run lint:fix --quiet 2>/dev/null || true"
        }]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{
          "type": "command",
          "command": "echo "$CLAUDE_TOOL_INPUT" | grep -qE '(rm -rf|DROP TABLE|truncate)' && exit 2 || exit 0"
        }]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [{
          "type": "command",
          "command": "osascript -e 'display notification "Claude is done" with title "Claude Code"' 2>/dev/null || notify-send 'Claude Code' 'Task complete' 2>/dev/null || true"
        }]
      }
    ]
  }
}
```

> **Boris's tip:** Boris's team uses a `PostToolUse` hook to auto-format Claude's code. Claude generates well-formatted code 90% of the time; the hook catches edge cases and prevents CI failures. **Strategy: block at commit (PostToolUse on the Bash/commit action), not at write** — blocking mid-plan confuses the agent.

### Slash Commands — Reusable Workflows

Custom slash commands live in `.claude/commands/` as Markdown files and are shared with the team.

```
.claude/commands/
├── review-pr.md       → /review-pr
├── deploy-staging.md  → /deploy-staging
├── fix-ci.md          → /fix-ci
└── update-docs.md     → /update-docs
```

**Example: `.claude/commands/review-pr.md`**
```markdown
---
description: Review the current PR for correctness, security, and style
---
Review the staged changes in this PR:
1. Check for logic errors and edge cases
2. Identify any security vulnerabilities (injection, auth bypass, data exposure)
3. Verify that tests cover the new code paths
4. Check adherence to CLAUDE.md coding standards
5. Write a structured review with: Summary, Issues (severity: high/medium/low), Suggestions
Grill me on the changes before approving — ask me at least 3 probing questions.
```

> **Boris's tip:** Use slash commands for workflows you repeat **many times a day**. Boris's subagent commands automate entire PR workflows. Commands can be invoked by Claude itself during agentic runs.

### Agents — Specialised Sub-Workers

Custom agents live in `.claude/agents/` and can be spawned by Claude or invoked directly.

```
.claude/agents/
├── security-reviewer.md
├── test-writer.md
└── db-migration-writer.md
```

**Example: `.claude/agents/test-writer.md`**
```markdown
---
name: test-writer
description: Writes comprehensive test suites for code changes
model: claude-sonnet-4-6
tools:
  - Read
  - Write
  - Bash(pnpm test *)
---
You are a specialist test engineer. When given code, you:
1. Identify all testable behaviours (happy path, error cases, edge cases)
2. Write Vitest tests following the project conventions in CLAUDE.md
3. Run the tests and fix failures
4. Achieve ≥80% code coverage on the changed files
Never modify production code. Only create or modify test files.
```

> Add `isolation: worktree` to agent frontmatter for parallel work without conflicts (Boris's February 2026 update).

### Skills — Reusable Knowledge Files

Skills are markdown knowledge files that Claude references automatically or on-demand.

```
~/.claude/skills/           ← personal, all projects
.claude/skills/             ← project-specific, team-shared
```

> Install Boris's skill: `npm install -g @anthropic-ai/claude-code` then type `/boris` to surface all 40 tips.

### Auto Memory — Claude's Own Notes

Claude automatically writes learnings to `~/.claude/projects/<project>/memory/`. The first 200 lines of `MEMORY.md` are injected at every session start; topic files are read on demand.

```
~/.claude/projects/my-project/memory/
├── MEMORY.md              ← index, auto-loaded (max 200 lines)
├── debugging.md           ← patterns Claude discovered
├── api-conventions.md     ← API decisions Claude learned
└── build-quirks.md        ← build system gotchas
```

**Enable or disable:**
```bash
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=0   # Force ON
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=1   # Force OFF
```

> You can tell Claude directly: **"Remember that we use pnpm, not npm"** — it writes to auto memory immediately. Run `/memory` to open the file selector and edit entries directly.

---

## Making It Project-Specific: Step-by-Step

Follow this sequence when setting up a new project:

### Step 1: Bootstrap
```bash
cd your-project
claude
> /init
```
Review and curate the generated `CLAUDE.md` — delete generic content, add project-specific truths.

### Step 2: Create the Directory Structure
```bash
mkdir -p .claude/rules .claude/commands .claude/agents
touch CLAUDE.local.md
echo "CLAUDE.local.md" >> .gitignore
```

### Step 3: Populate Level 3 (Project Memory)
Fill `CLAUDE.md` (or `.claude/CLAUDE.md`) using the Level 3 template above. Focus on:
- `pnpm dev` / `pnpm test` / `pnpm build` — exact commands with flags your team actually uses
- Real architecture (not aspirational)
- Actual gotchas you've already hit
- Your MCP servers (Supabase, Slack, GitHub, etc.)

### Step 4: Extract Rules to Level 4
Move domain-specific rules into `.claude/rules/`:
- One file per concern (testing, API, database, security, frontend)
- Add `paths:` frontmatter to scope rules to relevant file types
- Keep each file under 50 lines; quality over quantity

### Step 5: Create Your settings.json
```bash
touch .claude/settings.json
```
Start with the starter template above. Add to `permissions.allow` any commands your CI runs automatically. Add to `permissions.deny` any files that should never be read (`.env`, `secrets/`).

### Step 6: Add Starter Hooks
Add the PostToolUse auto-format hook and the Stop notification hook. Commit both to `.claude/settings.json`.

### Step 7: Create Your First Commands
```bash
touch .claude/commands/review-pr.md
touch .claude/commands/fix-ci.md
```

### Step 8: Populate CLAUDE.local.md
Add your local URLs, test data identifiers, and any active investigation context.

### Step 9: Iterate with Boris's Loop
After every correction session:
> **"Update CLAUDE.md so you don't make that mistake again."**

After every PR merge, tag `@.claude` to have Claude propose new `CLAUDE.md` learnings.

---

## Anti-Patterns: What NOT to Do

| Anti-Pattern | Why It Fails | Fix |
|---|---|---|
| Generic advice ("write good code") | Claude can't act on it | Be specific: "Use 2-space indentation, single quotes" |
| Stale CLAUDE.md never updated | Claude learns wrong patterns | Treat CLAUDE.md like living code; review monthly |
| Huge monolithic CLAUDE.md | Wastes tokens; hard to maintain | Split into Level 4 rules using `@imports` |
| Project config in User memory | Breaks other projects | Move to `./CLAUDE.md` or `CLAUDE.local.md` |
| Secrets in any CLAUDE file | Security risk even if gitignored | Use `.env.local` + reference var names only |
| `--dangerously-skip-permissions` in shared env | Unsafe, unchecked | Use `sandbox` mode or pre-allow specific commands via `/permissions` |
| Blocking hooks on file Write/Edit | Confuses mid-plan agent | Block at commit/submit time instead |
| No Plan Mode for complex tasks | Costly rework downstream | Start every complex session with Shift+Tab twice |
| Pushing when things go sideways | More confusion, more tokens | Switch back to Plan Mode; re-plan before continuing |

---

## Boris's Top Workflow Patterns (Applied to Any Project)

1. **Parallel worktrees:** Run `claude --worktree` for each major feature; name worktrees by task
2. **Always start in Plan Mode:** Shift+Tab twice; iterate the plan until solid, then auto-accept
3. **After a mediocre fix:** Say *"Knowing everything you know now, scrap this and implement the elegant solution"*
4. **Challenge Claude:** *"Grill me on these changes; don't make a PR until I pass your test"*
5. **Let Claude self-correct:** After every correction: *"Update CLAUDE.md so you don't make that mistake again"*
6. **Use subagents for context hygiene:** *"Use subagents"* appended to any request keeps main context clean
7. **Specs before hand-off:** Write a detailed spec and reduce ambiguity before handing off to auto-mode
8. **Loop closure is non-negotiable:** Always give Claude a way to verify its own output (test suite, browser, simulator)
9. **Slack MCP for bug reports:** Paste a Slack bug thread and say *"fix"* — zero context switching
10. **Compounding Engineering:** Every correction → CLAUDE.md update; every PR → @.claude review → CLAUDE.md PR

---

*Last updated: February 2026 | Maintained as a living document*
