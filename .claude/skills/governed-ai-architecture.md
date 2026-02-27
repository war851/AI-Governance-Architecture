---
name: governed-ai-architecture
description: Design methodology skill. Use when architecting an AI application where the database owns the process and the LLM is the semantic engine. Applies the same separation-of-concerns principle that makes Claude Code's configuration cascade work — but to a database-driven application. Reference before writing any code — design the governance model first.
---

# Governed AI Architecture

The same principle that makes Claude Code effective — structure beats intelligence — applied to AI applications. Don't train your AI. Certify it. Load skills at runtime, store everything, make every decision traceable.

---

## Quick Reference: The Architecture

```
┌──────────┐
│ FRONTEND │  Chat UI
└────┬─────┘
     │
     ▼
┌──────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR                            │
│                 (state machine, not an LLM)                │
│                                                           │
│  1. Read current step                                     │
│  2. Load linked: message · edge function · skill(s)       │
│  3. Build dynamic prompt from DB state                    │
│  4. Execute: code | code+LLM | sub-agents in parallel    │
│  5. Store: input as-is · output structured · log          │
│  6. Advance → next step (data evolution)                  │
│                                                           │
│  Every LLM call: TDD tested with golden examples          │
└────┬─────────────────────────────────────────────────────┘
     │
     ▼
┌──────────────────────────────────────────────────────────┐
│                    SINGLE DATABASE                         │
│         local · LLM-agnostic · configurable · transparent  │
│                                                           │
│  CONFIG        steps · messages · edge functions · skills  │
│  DATA          datasets (grow step by step)                │
│  TRANSPARENCY  chat transcript · audit log · execution log │
│  FILES         buckets (uploads, reports)                  │
│  SUB-AGENTS    each owns: skill config + dataset           │
└──────────────────────────────────────────────────────────┘
```

---

## The Bridge: Claude Code → DB Governed App

If you understand how Claude Code's configuration makes AI coding effective, you already understand this architecture. The same separation of concerns, applied to a database.

| Claude Code | DB Governed App | Same Principle |
|---|---|---|
| **CLAUDE.md cascade** — 5 levels define behaviour | **Config tables** — steps, messages, skills define behaviour | Behaviour lives in config, not in code or weights |
| **Skills files** — methodology loaded at runtime | **Skills table** — methodology loaded at runtime | Don't train — certify. Swap a skill, change the lens |
| **Slash commands** — executable units triggered by name | **Edge functions** — executable units triggered by step | Defined input → defined logic → defined output |
| **Rules** — path-scoped, loaded when relevant | **Step prerequisites + visibility** — scoped, loaded when relevant | Right context at the right moment. Nothing more |
| **PLAN.md** — explicit sequence of work | **Steps table** — explicit ordered sequence | Process is defined, not improvised by the LLM |
| **TODO.md** — tracks what feeds what | **Data evolution** — output of step N = input of step N+1 | The value chain of information is explicit |
| **Auto memory** — persisted across sessions | **Chat transcript + audit log** — persisted, queryable | Nothing lost in the context window |
| **Agents** — independent sub-workers | **Sub-agents** — parallel skill execution | Each owns its workspace, orchestrator coordinates |
| **settings.json** — permissions, allowed tools | **Function registry** — type, I/O schema, linked skill | What's allowed, what type, what contract |
| **Worktrees** — isolated workspaces per task | **Sub-agent dataset tables** — isolated data per skill | Workers don't share mutable state |
| **Git history** — full version trail | **Audit log + versioned datasets** — full trace | Every decision reconstructable |
| **Hooks** — pre/post gates around actions | **Orchestrator loop** — before/after gates per step | Automated quality control around every execution |

**The punchline is identical:** You don't make Claude Code better by making the LLM smarter. You make it better by configuring CLAUDE.md, rules, skills, and commands. You don't make a governed AI app better by training the model. You make it better by configuring steps, skills, functions, and messages in the DB.

---

## The Principle

Six rules. These are the constitution — like the managed policy layer in Claude Code, they cannot be overridden.

1. **DB owns the process, LLM is the semantic engine.** The database defines every step, every message, every function, every skill. The LLM receives a constrained prompt and returns structured output. It never decides what happens next.

2. **Don't train — certify.** Skills are runtime config, like certification for humans. The LLM follows the loaded framework. Swap a skill row, change the methodology. No retraining, no redeployment.

3. **Data evolution is the value chain.** The output of step N is the input of step N+1. Map this chain before writing any code — it becomes your steps, your schema, your functions, your storage.

4. **Every LLM call is constrained and testable.** Expected input + detailed prompt + skill config = structured output. Every call gets TDD tested with golden examples. If you can't write the test, you haven't defined the function.

5. **Everything is stored.** Input as-is, output structured, full audit trail. Chat transcript, execution log, prompt log. Nothing lives only in the context window.

6. **LLM-agnostic.** Swap the provider, nothing else changes. The governance model is in the DB, not in the model.

---

## The Components

Each component maps to something you already know from Claude Code. One responsibility each. Blur the boundaries and you lose governance.

### Steps → like PLAN.md

One unit of work. Ordered, with prerequisites. Can be visible (user-facing) or invisible (auto-execute). The steps table IS your process — change a row, change the flow.

✅ Step has one transformation, defined completion criteria, and explicit prerequisites — like a PLAN.md phase with a clear exit condition.

❌ Step does three things at once, or has no defined completion — like a PLAN.md that says "do the work" with no checkpoints.

### Static Messages → like CLAUDE.md display rules

All text the user sees. Stored in the DB, never hardcoded. Change a row, change what the user sees — like editing a static message in CLAUDE.md without redeploying.

✅ Every user-facing string comes from the messages table. Tone, language, instructions — all configurable.

❌ Display text hardcoded in function logic — requires a code change to fix a typo.

### Edge Functions → like slash commands

The executable unit linked to a step. Three types, just like Claude Code has different command complexities:

| Type | Claude Code Analogy | When |
|---|---|---|
| **Pure code** | Simple bash command | No semantic understanding needed |
| **Code + LLM** | Slash command that calls Claude | Semantic flexibility with structured I/O |
| **Sub-agents** | Parallel agents in worktrees | Multiple skills process the same input independently |

✅ Function has a defined input schema, defined output schema, linked skill (if LLM), and is independently testable — like a slash command with a clear spec.

❌ Function chains multiple LLM calls internally so you can't test or trace any single call — like a slash command that does 5 things and you can't tell which one failed.

### Skills → like skills files

A methodology loaded at runtime from the DB. The function defines WHAT to do; the skill defines THROUGH WHAT LENS. Like Claude Code skills files: same agent, different expertise depending on which skill is loaded.

Build skills with a standard structure that works across domains — same config shape, different content. A compliance skill and a risk assessment skill share the same format; only the dimensions, criteria, and rubrics differ. Tune for any domain by loading the right knowledge, not by rebuilding the config.

✅ Skill references a real framework, defines specific dimensions, requires evidence for every rating, and uses the same config structure as every other skill — swappable, testable, domain-agnostic format.

❌ Skill dimensions don't match the source methodology (you skipped the research), or output captures scores without evidence (you can't audit the reasoning), or two frameworks are jammed into one skill instead of running as parallel sub-agents.

### Sub-Agents → like agents in worktrees

When multiple skills must process the same input independently, each becomes a sub-agent. Each owns its own skill config AND its own dataset table — like Claude Code agents in isolated worktrees. The orchestrator invokes all of them and collects results. They never share mutable state.

✅ Three sub-agents each own their config and dataset. Orchestrator runs them in parallel, collects results, merges in a separate step.

❌ Sub-agents write to the same table — race conditions, unclear ownership, untraceable results.

### Storage → like git + auto memory

Everything is stored. Raw and processed. The DB is your git history — every decision is reconstructable.

| What | Where | Claude Code Analogy |
|---|---|---|
| Process definition | Config tables | CLAUDE.md + rules + settings.json |
| Structured outputs | Dataset tables | Generated code + artifacts |
| Conversation | Chat transcript (with step_id) | Terminal session history |
| Every LLM call | Audit log (prompt + response + tokens) | Git commits with full diffs |
| Every function call | Execution log | CI/CD pipeline logs |
| Raw files | Buckets | Working directory files |

✅ Input stored as-is, output stored structured, both kept, every record tagged with step_id — full traceability.

❌ Relying on context window as memory, overwriting without versioning, storing only final results — like coding without git.

### Orchestrator → like Claude Code's state machine

A state machine, not an LLM. It reads the current step, loads everything linked to it, assembles the prompt from DB state, executes, stores, advances. It never decides — it coordinates. Like Claude Code's intent → plan → execute → validate → trace cycle.

**The orchestrator loop:**
```
1. Get/create session
2. Persist user message to chat transcript
3. Load current step
4. If invisible → auto-execute → store → advance → repeat
5. If visible → load message + function + skill + data + files + history
   → assemble dynamic prompt → single stateless LLM call
   → parse response → store input + output + audit
   → if complete → advance | else → stay, re-prompt next message
```

**The dynamic prompt** is assembled from DB state in this order:
```
1. Runtime identity       (who you are)
2. Current step           (from steps table)
3. Static message         (from messages table)
4. Function logic         (from functions table)
5. Skill config           (from skills table)
6. Step data              (from dataset tables — outputs of previous steps)
7. Bucket files           (relevant uploads)
8. Chat transcript        (conversation history)
9. Expected output schema (the contract)
```

Every prompt is reconstructable from the DB state at that moment — like being able to check out any git commit and see exactly what Claude Code saw.

---

## Anti-Patterns: Governance Violations

Every anti-pattern is a boundary that got blurred — the same category of mistake as vibe coding instead of governing.

| Violation | What Got Blurred | Fix |
|---|---|---|
| LLM decides what step to execute next | Process ownership (DB → LLM) | Steps table defines order; orchestrator advances |
| Methodology baked into prompts | Skill boundary (config → code) | Move to skills table — configurable, swappable |
| Relying on context window as memory | Storage boundary (DB → ephemeral) | Store everything: datasets, chat, audit log |
| One giant prompt for the whole process | Step isolation (scoped → global) | One constrained prompt per step from DB state |
| Functions without defined I/O | Contract boundary (explicit → implicit) | Every function has input schema + output schema |
| Sub-agents sharing tables | Workspace isolation (owned → shared) | Each sub-agent owns its dataset table |
| Skipping the data evolution map | Value chain (planned → improvised) | Map the chain first: raw → each transform → final |
| Multi-step logic in one edge function | Testability (isolated → entangled) | One LLM call per function; split compound logic |
| Display text hardcoded in functions | Message boundary (config → code) | Static messages table — change a row, change the text |
| Training instead of configuring skills | Governance model (runtime → frozen) | Skills are config — add, change, remove instantly |
| Skill structure varies per domain | Reusability (standard → bespoke) | Same config shape, different content. Tune with knowledge |

---

## Validation Checklist

Before considering the architecture complete:

- [ ] Data evolution mapped end-to-end (raw input → each transformation → final outcome)?
- [ ] Every step: one responsibility, defined completion, explicit prerequisites?
- [ ] Every edge function: defined I/O schema, function type, independently testable?
- [ ] Every skill: config in DB, references real framework, standard reusable structure?
- [ ] Every LLM call: golden examples for TDD testing?
- [ ] Storage: input as-is + output structured + audit trail + step_id on everything?
- [ ] Orchestrator: state machine, LLM-agnostic, never decides — only coordinates?
- [ ] Static messages: all user-facing text in DB, zero hardcoded strings?
- [ ] Sub-agents: each owns config + dataset, no shared mutable state?
- [ ] Dynamic prompt: assembled from DB state, reconstructable from any point in time?

---

## Artifact Governance

When creating artifacts from this skill:

1. Use the templates in `.claude/skills/artifact-template.md` — copy the relevant section (steps-table, edge-functions, skills-config, static-messages, or orchestrator-design)
2. Fill in the YAML frontmatter with project, version, status, and source
3. Register the artifact in `DESIGN.md` (project root) — add a row to the Artifact Registry table

---

## Referenced Skills

| Skill | When to Use |
|-------|-------------|
| **Value Chain Mapping** | Before designing — map the data evolution chain |
| **TDD Golden Examples** | Testing — golden examples for every LLM call |

---

*Structure beats intelligence. Configure, don't train. The DB governs. The LLM serves.*
