# Governed AI — Workspace Anatomy

> A complete walkthrough of every file in this workspace, what it does, and how it connects to the whole.
> This is the "read this first" document for anyone exploring the repository.

---

## The Big Picture

This workspace is a **project-agnostic starter template** for building AI applications using a governed architecture pattern. It configures two AI agents — **Cursor** (IDE) and **Claude Code** (terminal) — to work together with clear separation of concerns, and provides four application design skills that take you from stakeholder interview to running application.

The core thesis: **structure beats intelligence.** Don't train your AI — certify it. Load skills at runtime, store everything, make every decision traceable.

---

## Root Governance Files

Three files at the project root form the governance backbone.

### `CLAUDE.md` — Team-Shared Project Memory

The primary configuration file for Claude Code. Checked into git, shared with the team.

**Contains:**
- Architecture overview — dual-agent model (Cursor + Claude Code + shared layer)
- Key directory map (`.claude/`, `.cursor/rules/`, `docs/`, `src/`)
- Coding standards — explicit over implicit, no silent catches, no hardcoded secrets
- Planning discipline — Boris's "plan first" pattern, phase-based execution
- Compounding engineering — update this file after every correction session
- Git workflow — Conventional Commits, branch naming, PR requirements
- Known gotchas — Windows-specific traps (PowerShell syntax, path separators, line endings)
- MCP tools and configuration references

**Key principle:** This file is a living document. It compounds with every interaction — every mistake taught, every pattern learned gets encoded here so it never recurs.

### `CLAUDE.local.md` — Personal Local Overrides

Auto-gitignored. Personal to the developer's machine.

**Contains:**
- Local environment specifics (OS, shell, editor, git config)
- Active investigation notes (cleaned up when done)
- Personal shortcuts ("plan it" → enter planning mode, "ship it" → commit and push)
- Experimental rules being tested before promotion to `CLAUDE.md`

### `DESIGN.md` — Application Design Governance Index

The equivalent of `CLAUDE.md` but for application design, not code.

**Contains:**
- Project metadata and status
- Skills-in-use table — which of the four skills are active and how far along
- Artifact registry — every design artifact, its type, version, and status
- Skill connections — how VCM feeds DE, TDD, and GA; how they feed back
- Template references and handoff criteria

**Key principle:** When all Governed Architecture artifacts reach `confirmed` status, they become the input for `PLAN.md` and implementation. DESIGN.md is the single index — read it to know where the project stands.

---

## Claude Code Configuration (`.claude/`)

Everything Claude Code needs to behave correctly: permissions, automation, rules, commands, agents, and skills.

### `.claude/settings.json` — Permissions, Hooks, Environment

The behaviour layer — controls what Claude Code is allowed to do and what happens automatically.

**Permissions:**
- **Allow:** Safe git commands (`status`, `diff`, `log`, `branch`, `checkout`, `add`, `commit`, `stash`, `fetch`, `pull`), reading any file
- **Deny:** Reading secrets (`.env`, `.env.*`, `secrets/`, `.aws/`, `*.pem`, `*.key`), destructive commands (`rm -rf`, `git push --force`, `git reset --hard`)
- **Default mode:** `acceptEdits` — Claude can edit files without confirmation

**Hooks:**
- **PreToolUse (Bash):** PowerShell guard that blocks dangerous patterns (`rm -rf`, `DROP TABLE`, `TRUNCATE`, `--force`, `--hard`) before execution — exits with code 2 to reject
- **Stop:** Plays a system beep (800Hz, 300ms) when Claude finishes a task — the Windows-compatible equivalent of a desktop notification

**Environment:**
- Telemetry disabled (`CLAUDE_CODE_ENABLE_TELEMETRY: 0`)

### `.claude/agents/` — Specialised Sub-Workers

Two read-only agents that can be spawned for specific tasks without polluting the main context.

#### `code-reviewer.md`
- **Tools:** Read, `git diff`, `git log`
- **Behaviour:** Reads all changed files, checks against `CLAUDE.md` standards, identifies security vulnerabilities and logic errors, produces a structured review with severity ratings, asks at least 3 challenging questions
- **Constraint:** Never modifies code — read and report only

#### `planner.md`
- **Tools:** Read, `git status`, `git log`
- **Behaviour:** Analyses the codebase, identifies affected files, creates a 3–7 phase implementation plan with dependencies, completion criteria, and risk assessment
- **Constraint:** Never writes production code — read, analyse, and plan only

### `.claude/commands/` — Slash Commands

Three reusable workflows invoked with `/command`.

#### `/plan` — Structured Planning
1. Restate the goal
2. Identify constraints
3. Map dependencies
4. Design 3–7 phases with scope, completion criteria, and complexity
5. Flag risks
6. Present and wait for approval before executing

#### `/review` — Code Review
1. Summarise what changed and why
2. Logic check for errors and edge cases
3. Security scan (hardcoded secrets, injection vectors, missing validation)
4. Style check against `CLAUDE.md` standards
5. Test coverage gap analysis
6. Completeness check (TODOs, partial implementations, dead code)
7. Structured verdict with severity ratings and probing questions

#### `/learn` — Compounding Engineering Loop
1. Identify the lesson (what went wrong or was discovered)
2. Categorise it (coding standard, gotcha, workflow, architecture)
3. Write a clear, actionable rule
4. Place it correctly (`CLAUDE.md` for global, `.claude/rules/` for scoped, `CLAUDE.local.md` for personal)
5. Confirm the update

### `.claude/rules/` — Modular Behavioural Rules

Four rule files loaded at every Claude Code session start. Each follows a **Requirements (MUST) / Conventions (SHOULD) / Anti-Patterns (NEVER)** structure.

#### `code-quality.md`
- Single responsibility per function
- Explicit error handling — no empty catches
- Parameterised queries for all DB operations
- Input validation on all public APIs
- Environment variables for all config
- Anti-patterns: no `any` types, no suppressed linters, no `TODO` without issue numbers, no `console.log` in production, no commented-out code

#### `git-workflow.md`
- Conventional Commits format with scope
- Atomic commits — one logical change each
- Branch naming: `feat/`, `fix/`, `chore/`
- Anti-patterns: no force-push to shared branches, no committed secrets, no generated files, no rewritten published history, no merge commits on feature branches

#### `planning-discipline.md`
- Plan before coding for 3+ file tasks or architectural decisions
- Numbered phases with completion criteria and verification steps
- Stop and re-plan on failure — never push forward on a broken plan
- Ask clarifying questions when uncertain
- Anti-patterns: no "just hacking" on complex features, no continuing a failing plan, no skipping verification

#### `windows-environment.md`
- PowerShell syntax for all shell operations
- Forward slashes in config files, backslashes only in native commands
- `2>$null` instead of `2>/dev/null`
- `Get-ChildItem` instead of `ls -la`, `Remove-Item` instead of `rm -rf`
- Anti-patterns: never assume bash, never use Unix-only utilities, never use symlinks without developer mode

### `.claude/skills/` — Knowledge Files

Six skills: one operational (configure-claude-code), four application design skills, and one artifact template reference.

#### `configure-claude-code.md` — Meta-Skill

The skill for configuring Claude Code itself. Contains an intent routing table that maps user requests to action paths:

- **Governed application patterns:** Adding steps, functions, messages to a table-driven architecture
- **Skills/agents/automation:** Creating new skills, agents, hooks, slash commands
- **Testing/quality:** TDD, security, access control
- **External services:** GitHub CLI
- **Project bootstrap:** 8-step checklist from CLAUDE.md to iteration

Also includes the 5-level cascade quick reference and a validation checklist for any change.

#### `value-chain-mapping.md` — Discovery & Interview Skill

Maps the process before designing the solution. Conducts structured interviews to trace every step from raw input to final deliverable.

**The drill-down principle:**
- Level 0: "What are we building?" → one-liner purpose
- Level 1: "How does it work end to end?" → 3–7 rough steps
- Level 2+: For each step → intention, input, output, completion criteria, love, hate

**Produces 4 artifacts:**
1. Value Chain Map — ordered step sequence
2. Step Specifications — one per step, growing from sparse to detailed
3. Stakeholder Register — who contributed what
4. Gap Log — what's unknown, shrinks to zero as discovery progresses

**Key constraint:** Stay in the problem domain. Never ask about technology or implementation — that constrains the architect.

#### `data-evolution.md` — Design & Visualization Skill

Traces how data transforms at the field level, not the step level. Runs alongside or after Value Chain Mapping.

**Core approach:** For each piece of data, follow its journey — where it enters (raw form), what happens to it (parsed, split, enriched), what it becomes (structured output), and why each transformation exists.

**Produces 5 artifacts:**
1. Data Evolution Diagram (Mermaid) — visual flow of transformations
2. Data State Diagram (Mermaid) — field shapes at each stage
3. Data Evolution Table — field-level lineage
4. Transformation Audit — is each transformation necessary?
5. Schema Derivation — tables and columns that emerge from the flow

**Key principle:** Don't design the database first. Let the data evolution tell you what tables you need. The schema is a consequence of how data needs to flow.

**Side benefits:** GDPR compliance (know where personal data lives), RLS design (know who owns each field), EU AI Act traceability, auditability — all for free when you understand your data at field level.

#### `tdd-golden-examples.md` — Testing Methodology Skill

Test-Driven Development adapted for governed AI applications. Every function gets a test before it gets an implementation.

**Red-Green-Refactor adapted:**
- **Red:** Write golden examples with evaluation criteria — no implementation exists yet
- **Green:** Write code + prompt + skill config that meets the criteria
- **Refactor:** Refine prompt, adjust skill config — golden examples must still pass

**Evaluation criteria types:** Completeness, faithfulness, structure, validity, relevance, consistency, absence (forbidden content checks).

**Key difference from traditional TDD:** For LLM-involving functions, "Green" means evaluation criteria are satisfied (right structure, required elements present, faithful to input) — not exact string matching.

**Testing hierarchy:**
- Pure code → standard unit tests with exact assertions
- Code + LLM → golden examples with criteria-based evaluation
- Sub-agents → test each independently + test the merge separately

**Produces 3 artifacts:**
1. Golden Example Sets — one per function with happy path, edge cases, failure modes
2. Test Specifications — how to run, what passes, isolation rules
3. Evaluation Rubrics — criterion, weight, pass/fail definitions

#### `governed-ai-architecture.md` — Design Methodology Skill

The architecture pattern itself. Applies the same separation of concerns that makes Claude Code work — but to a database-driven application.

**The bridge (Claude Code → DB Governed App):**

| Claude Code | DB Governed App | Same Principle |
|---|---|---|
| CLAUDE.md cascade | Config tables (steps, messages, skills) | Behaviour lives in config, not in code or weights |
| Skills files | Skills table | Don't train — certify. Swap a skill, change the lens |
| Slash commands | Edge functions | Defined input → defined logic → defined output |
| Rules (path-scoped) | Step prerequisites + visibility | Right context at the right moment |
| PLAN.md | Steps table | Process is defined, not improvised by the LLM |
| TODO.md | Data evolution | Output of step N = input of step N+1 |
| Auto memory | Chat transcript + audit log | Nothing lost in the context window |
| Agents | Sub-agents | Each owns its workspace, orchestrator coordinates |
| settings.json | Function registry | What's allowed, what type, what contract |
| Git history | Audit log + versioned datasets | Every decision reconstructable |
| Hooks | Orchestrator loop | Automated quality control around every execution |

**6 Constitutional Principles:**
1. DB owns the process, LLM is the semantic engine
2. Don't train — certify
3. Data evolution is the value chain
4. Every LLM call is constrained and testable
5. Everything is stored
6. LLM-agnostic

**Components:** Steps (like PLAN.md), Static Messages (like CLAUDE.md display rules), Edge Functions (like slash commands — pure code / code+LLM / sub-agents), Skills (like skills files — methodology loaded at runtime), Sub-Agents (like agents in worktrees — each owns config + dataset), Storage (like git + auto memory), Orchestrator (state machine, not an LLM).

**The orchestrator loop:**
1. Get/create session
2. Persist user message
3. Load current step
4. If invisible → auto-execute → store → advance → repeat
5. If visible → load everything → assemble dynamic prompt → single LLM call → parse → store → advance or stay

**Produces 5 artifacts:**
1. Steps Table — ordered process with prerequisites and function types
2. Edge Functions — typed executables with I/O schemas
3. Skills Config — runtime methodology, swappable
4. Static Messages — all user-facing text in DB
5. Orchestrator Design — state machine loop specification

#### `artifact-template.md` — Template Reference

Contains the generic YAML frontmatter envelope and content templates for all **17 artifact types** across the four skills:

- **VCM (4):** value-chain-map, step-spec, stakeholder-register, gap-log
- **DE (5):** evolution-diagram, state-diagram, evolution-table, transformation-audit, schema-derivation
- **TDD (3):** golden-example-set, test-spec, eval-rubric
- **GA (5):** steps-table, edge-functions, skills-config, static-messages, orchestrator-design

Every artifact is self-describing via YAML frontmatter: name, artifact_type, skill, project, purpose, version, status (`draft` → `detailed` → `confirmed`), date, source, step reference, and linked artifacts.

---

## Cursor Configuration (`.cursor/rules/`)

Three always-apply rules in `.mdc` format that Cursor loads at every session.

### `project-conventions.mdc`
- Agent separation of concerns (Cursor owns code standards; Claude Code owns planning/execution; shared layer owns CLAUDE.md, plans, docs)
- Code quality standards (explicit over implicit, single responsibility, no `any` types, no commented-out code)
- Naming conventions (kebab-case files, camelCase variables, PascalCase classes, UPPER_SNAKE_CASE constants, boolean prefixes)
- File organization (group by feature, co-locate tests, 300-line limit, one export per file)
- Environment notes (Windows 10, PowerShell, forward slashes in config)

### `planning-and-execution.mdc`
- When to plan first (3+ files, architectural decisions, schema changes, anything "complex")
- Plan structure (restate goal, identify constraints, numbered phases, flag risks, wait for approval)
- Execution rules (one phase at a time, stop and re-plan on failure, loop closure)
- Compounding engineering (update config after corrections)

### `git-standards.mdc`
- Conventional Commits format
- Branch naming patterns
- Forbidden actions (no force-push, no committed secrets, no generated files, no rewritten history)

---

## Documentation (`docs/`)

### `claude-code-configuration-guide.md` — The 5-Level Cascade

A comprehensive guide (~35K chars) covering every layer of Claude Code configuration with templates, positive examples, negative examples, limits, and Boris best practices.

**The 5 levels (highest precedence → lowest):**
1. **Managed Policy** (`/etc/claude-code/CLAUDE.md`) — org-wide, IT-deployed, non-overridable
2. **User Memory** (`~/.claude/CLAUDE.md`) — personal, all projects
3. **Project Memory** (`./CLAUDE.md`) — team-shared, git-committed
4. **Project Rules** (`.claude/rules/*.md`) — modular, path-scoped
5. **Local Override** (`./CLAUDE.local.md`) — personal, gitignored

**Plus:** settings.json (permissions, hooks), skills (on demand), agents (on spawn), commands (on invoke), auto memory (first 200 lines).

**Also covers:**
- settings.json scopes (user, project, local)
- Hooks (PreToolUse, PostToolUse, Stop, SessionStart) with starter configs
- Slash commands with examples
- Agents with frontmatter spec
- Skills directories
- Auto memory system
- Step-by-step project setup (8 steps)
- Anti-patterns table
- Boris's top 10 workflow patterns

### `claude-code-configuration-sources.md` — Reference URLs

Canonical source list for extending the configuration:
- Anthropic official docs (Memory, Settings, Hooks guide, Hooks reference)
- Anthropic engineering blog (agentic coding best practices, CLAUDE.md blog post)
- Boris's site (all 4 threads, 40 tips)
- Decision tree for which source to use when

---

## Interactive Visualization (`docs/visualizations/`)

A single interactive HTML file — `Interactive Governed AI Architecture.html` — serves as both HLD and LLD:

- **HLD view** — the high-level architecture: DB tables (steps, functions, skills, messages), the Claude Code ↔ DB bridge, and the 6 constitutional principles
- **LLD view** — the skill ecosystem with four sub-views: The Pyramid (skill hierarchy), The Flow (artifact pipeline), The Governance (DESIGN.md, artifact lifecycle), and The Constitution (principles)

Opens standalone in any browser. Dark theme, JetBrains Mono font, consistent colour palette: amber (#f59e0b) for VCM, purple (#a78bfa) for Data Evolution, pink (#f472b6) for TDD, emerald (#34d399) for Governed Architecture, blue (#60a5fa) for Claude Code Environment.

---

## Skill Connections — How Everything Feeds Everything

```
                    ┌─────────────────┐
                    │  Value Chain     │
                    │  Mapping (VCM)   │
                    └───┬───┬───┬─────┘
                        │   │   │
              ┌─────────┘   │   └─────────┐
              ▼             ▼             ▼
    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
    │    Data      │ │    TDD      │ │  Governed   │
    │  Evolution   │ │   Golden    │ │ Architecture│
    │    (DE)      │ │  Examples   │ │    (GA)     │
    └──────┬──────┘ └──────┬──────┘ └──────▲──────┘
           │               │               │
           └───────────────┴───────────────┘
                           │
                    ┌──────▼──────┐
                    │ Claude Code  │
                    │ Environment  │
                    └─────────────┘
```

**Forward flows:**
- VCM → DE (step specs provide the starting point for field-level tracing)
- VCM → TDD (step specs become golden example sources)
- VCM → GA (confirmed map is the architecture input)
- DE → TDD (field transformations need test coverage)
- DE → GA (schema derivation drives DB design)
- TDD → GA (golden examples become function contracts)

**Feedback loops:**
- DE → VCM (transformation issues loop back to step design)
- TDD failures refine prompts and skill configs
- Compounding engineering applies across all skills

---

## The Artifact Lifecycle

Every artifact follows the same progression:

```
draft ──────────► detailed ──────────► confirmed
intention         sections filled       validated and
captured,         through iterative     ready for downstream
sparse content    conversation          use or handoff
```

All artifacts are:
- Self-describing (YAML frontmatter with name, type, skill, version, status)
- Registered in `DESIGN.md` (single index for project state)
- Versioned (never overwrite — always version up)
- Linked (artifacts reference each other via `linked_artifacts`)

When Governed Architecture artifacts reach `confirmed`, they become the implementation spec for PLAN.md and code development.

---

## The Dual-Agent Model

| Concern | Owner | Location |
|---------|-------|----------|
| Code standards, naming, architecture patterns | **Cursor** | `.cursor/rules/*.mdc` |
| Planning, execution, skills, agents, hooks, commands | **Claude Code** | `.claude/` |
| Project memory, shared knowledge | **Both** | `CLAUDE.md`, `DESIGN.md`, `docs/` |
| Personal overrides | **Individual** | `CLAUDE.local.md` (gitignored) |

---

*Structure beats intelligence. Configure, don't train. The DB governs. The LLM serves.*
