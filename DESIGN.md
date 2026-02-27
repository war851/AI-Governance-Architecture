# DESIGN.md — Application Design Governance
> The index of all application design artifacts for this project.
> Equivalent of CLAUDE.md but for application design, not code.

---

## Project

- **Name:** [project name]
- **Status:** [discovery | design | testing | architecture | implementation]
- **Last updated:** [YYYY-MM-DD]

---

## Skills in Use

| Skill | Status | Artifacts Created |
|-------|--------|-------------------|
| Value Chain Mapping | [not started/in progress/confirmed] | [count] |
| Data Evolution | [not started/in progress/confirmed] | [count] |
| TDD Golden Examples | [not started/in progress/confirmed] | [count] |
| Governed Architecture | [not started/in progress/confirmed] | [count] |

---

## Artifact Registry

All artifacts for this project. Add a row when an artifact is created.

| Filename | Type | Skill | Step | Version | Status |
|----------|------|-------|------|---------|--------|
| [filename.md] | [artifact_type] | [vcm/de/tdd/ga] | [step or —] | [N] | [draft/detailed/confirmed] |

---

## Skill Connections

How skills feed each other:

- VCM → DE (step specs provide the starting point)
- VCM → TDD (step specs become golden example sources)
- VCM → GA (confirmed map is the architecture input)
- DE → TDD (field transformations need test coverage)
- DE → GA (schema derivation drives DB design)
- DE → VCM (feedback — transformation issues loop back to step design)
- TDD → GA (golden examples become function contracts)

---

## Templates

All artifact templates are defined in [artifact-template.md](.claude/skills/artifact-template.md).

---

## Handoff to Claude Code

When the Governed Architecture artifacts (steps-table, edge-functions, skills-config, static-messages, orchestrator-design) reach `confirmed` status, they become the input for PLAN.md and implementation.
