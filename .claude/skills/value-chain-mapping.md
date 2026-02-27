---
name: value-chain-mapping
description: Discovery and interview skill. Use before any architecture or implementation work to map the full process from first input to final output. Conducts structured interviews, drills down from intention to detail, and produces versioned artifacts that feed the governed architecture skill. Product, process, and technology agnostic.
---

# Value Chain Mapping

Map the process before designing the solution. Conduct structured interviews to trace every step from raw input to final deliverable — capturing intentions, inputs, outputs, what works, and what's painful. Produce artifacts that describe the problem faithfully, without prescribing implementation.

---

## How This Skill Works

### 1. Check Existing State

Before starting, check what already exists:
- PLAN.md, TODO.md, existing documentation
- Previous versions of the four artifacts (below)
- Any prior interview notes or stakeholder input

**Cold start** (nothing exists): Begin with "What are we building?"
**Warm resume** (artifacts exist): Read them, identify gaps, continue from where things left off.

### 2. Interview with Drill-Down

Conduct structured interviews using progressive depth. Every question stays in the **problem domain** — intentions, inputs, outputs, pain points. Never ask about implementation, technology choices, or architecture decisions.

**The drill-down principle:**

| Level | Question | Produces |
|-------|----------|----------|
| 0 | What are we building? | One-liner purpose |
| 1 | How does it work end to end? | High-level process flow (3–7 rough steps) |
| 2+ | For each step: What is the intention? What goes in? What comes out? What does "done" look like? What do you love? What do you hate? | Detailed step specifications |

Keep drilling into each step until the intention, input, output, and completion criteria are clear enough to hand off. If a step reveals sub-steps, expand the map.

**When the interviewee can't answer:**
- "Who would know this?" → Add to stakeholder register, create gap log entry
- "I don't know, I work alone" → "Want me to help you think through this part?" → Collaborative design, flag as design decision in gap log
- "That's not decided yet" → Log as open decision, continue with what IS known

### 3. Translate Every Conversation into Artifacts

Every interview, discussion, follow-up, or decision updates the four artifacts. Nothing stays only in conversation — it gets captured in structured form.

### 4. Check Completion

The discovery is complete when:
- The value chain map covers the full process from first input to final output
- Every step has a specification with intention, input, output, and completion criteria
- The gap log is empty or all remaining items are marked "design decision"
- At least one stakeholder has confirmed the map reflects reality

### 5. Hand Off

The confirmed artifacts become the input to the **Governed Architecture** skill, which uses them to design the DB-governed application architecture.

---

## The Drill-Down Questions

For each step in the process, ask in this order. Stay in the problem domain — never cross into implementation.

| Category | Questions |
|----------|-----------|
| **Intention** | What is this step trying to achieve? Why does it exist? What would go wrong if we skipped it? |
| **Input** | What does this step receive? Where does it come from? What form is it in? |
| **Output** | What does this step produce? Who or what consumes it? What form must it be in? |
| **Completion** | How do you know this step is done? What does a good result look like? What does a bad result look like? |
| **Love** | What works well about this today? What must we preserve? |
| **Hate** | What's painful, slow, broken, or frustrating? What do you wish was different? |

✅ "I want to verify the job description's requirements against evidence from the CV" — intention, clear, no technology prescribed.

❌ "This should use keyword matching with an LLM fallback" — implementation decision, premature, constrains the architect.

---

## Artifacts

Four artifacts, versioned together. They start sparse and fill in with each session. Every artifact is a living document — never overwrite, always version.

### 1. Value Chain Map

The ordered sequence from first input to final output. Each entry is one step in the process.

```markdown
# Value Chain Map
# Version: [N] | Last updated: [date] | Status: [draft/confirmed]

## Process: [name]
[One-sentence purpose]

| # | Step | Intention | Input | Output | Status |
|---|------|-----------|-------|--------|--------|
| 1 | [name] | [what it achieves] | [what it receives] | [what it produces] | [mapped/detailed/confirmed] |
| 2 | [name] | [what it achieves] | [what it receives] | [what it produces] | [mapped/detailed/confirmed] |
| ... | | | | | |

## Data Evolution
[First input] → Step 1 → [intermediate] → Step 2 → ... → [final output]
```

### 2. Step Specifications

One per step. Grows from sparse to detailed across sessions.

```markdown
# Step Specification: [Step Name]
# Version: [N] | Source: [who provided this information]

## Intention
[What this step is trying to achieve and why it exists]

## Input
- [What it receives, in what form, from where]

## Output
- [What it produces, in what form, for whom]

## Completion Criteria
- [How you know it's done — good result vs bad result]

## Love (preserve)
- [What works well about this today]

## Hate (solve)
- [What's painful, slow, broken, frustrating]

## Open Questions
- [Anything still unclear — links to gap log]
```

### 3. Stakeholder Register

Tracks who contributed what and who to talk to next.

```markdown
# Stakeholder Register
# Version: [N]

| Name | Role | Interviewed | Informed Steps | Gaps Identified | Follow-up With |
|------|------|-------------|----------------|-----------------|----------------|
| [name] | [role] | [date or pending] | [step numbers] | [what they couldn't answer] | [who they suggested] |
```

### 4. Gap Log

Tracks what's unknown. Drives the next interview. Shrinks to zero as discovery progresses.

```markdown
# Gap Log
# Version: [N] | Open: [count] | Closed: [count]

| # | Gap | Identified By | Step | Status | Resolution |
|---|-----|---------------|------|--------|------------|
| 1 | [what's unknown] | [stakeholder] | [step #] | [open/closed/design decision] | [how it was resolved, or who to ask] |
```

---

## Anti-Patterns

| Mistake | Why It Fails | Fix |
|---------|-------------|-----|
| Asking about technology or implementation | Constrains the architect with premature decisions | Stay in the problem domain: intention, I/O, pain |
| Skipping "love" and "hate" questions | Miss what to preserve and what to solve | These drive priorities more than any feature list |
| Interviewing only one role | Blind spots — sales sees different reality than engineering | Follow the gap trail to whoever has the missing piece |
| Creating a perfect map before starting | Analysis paralysis — the map is never "done" enough | Start sparse, version up, hand off when gaps are closed |
| Overwriting artifacts instead of versioning | Lose the evolution — can't see what changed or why | Always version, never overwrite |
| Mixing problem description with solution design | Artifacts become unusable for an unbiased architect | The map describes WHAT and WHY — never HOW |

---

## Validation Checklist

Before handing off to the Governed Architecture skill:

- [ ] Value chain map covers first input to final output with no missing links?
- [ ] Every step has a specification with intention, input, output, and completion criteria?
- [ ] Love and hate captured for steps where stakeholders had opinions?
- [ ] Data evolution chain is explicit — output of step N feeds step N+1?
- [ ] Stakeholder register documents who informed what?
- [ ] Gap log is empty or all items resolved as closed/design decision?
- [ ] At least one stakeholder has confirmed the map reflects current reality?
- [ ] Zero implementation or technology decisions in the artifacts?

---

## Artifact Governance

When creating artifacts from this skill:

1. Use the templates in `.claude/skills/artifact-template.md` — copy the relevant section (value-chain-map, step-spec, stakeholder-register, or gap-log)
2. Fill in the YAML frontmatter with project, version, status, and source
3. Register the artifact in `DESIGN.md` (project root) — add a row to the Artifact Registry table

---

## Referenced Skills

| Skill | Relationship |
|-------|-------------|
| **Data Evolution** | Peer — runs alongside or after; provides field-level depth for each step's I/O |
| **TDD Golden Examples** | Peer — uses the step specifications to derive testable golden examples |
| **Governed Architecture** | Downstream — receives the confirmed artifacts and designs the DB-governed application |
| **Configure Claude Code** | Foundation — the development environment where the designed application will be built |

---

*Map the process. Capture the intention. Describe the problem. Let the architect design the solution.*
