# Planning Discipline Rules

## Overview

Enforces Boris's Plan Mode pattern: think first, plan thoroughly, execute confidently.

## Requirements (MUST)

- For tasks touching 3+ files or involving architectural decisions: create a plan BEFORE writing code
- Every plan must have numbered phases with explicit completion criteria
- Each phase that changes behaviour must include a verification step (test, manual check, or both)
- When a plan fails mid-execution: STOP, re-enter Plan Mode, revise the plan before continuing
- After completing a task, verify the output matches the original specification

## Conventions (SHOULD)

- Break large features into phases that can each be verified independently
- Identify dependencies between phases upfront to avoid rework
- State assumptions explicitly at the start of the plan
- Include rollback strategy for destructive operations (migrations, data changes)
- Use subagents for independent subtasks to keep main context clean

## Anti-Patterns (NEVER)

- NEVER start coding a complex feature without a plan — "just hacking" wastes tokens and creates rework
- NEVER continue executing a failing plan — stop, re-plan, then resume
- NEVER skip the verification step in a phase — unverified changes compound errors
- NEVER push forward when uncertain — ask clarifying questions instead
