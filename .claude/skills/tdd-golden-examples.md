---
name: tdd-golden-examples
description: Testing methodology skill. Use when defining and validating any function in a governed architecture — especially code+LLM combinations. Derives testable golden examples from step specifications, applies the Red-Green-Refactor cycle adapted for semantic outputs, and ensures every function is independently verifiable before integration. Product, architecture, and technology agnostic.
---

# TDD with Golden Examples

Test-Driven Development adapted for governed AI applications. Every function — whether pure code, code+LLM, or sub-agent — gets a test before it gets an implementation. For LLM-involving functions, golden examples replace exact assertions with evaluation criteria that validate output quality without requiring deterministic string matching.

The principle: **if you can't write the test, you haven't defined the function.**

---

## How This Skill Works

### 1. Start from Step Specifications

This skill consumes the artifacts produced by the Value Chain Mapping skill. Each step specification already defines:
- **Intention** — what the step achieves
- **Input** — what it receives
- **Output** — what it produces
- **Completion criteria** — what "done" looks like

These four elements are everything you need to write a test.

### 2. Derive Golden Examples Before Writing Code

For each function linked to a step:
1. Take a real or realistic input (from the step specification or stakeholder interviews)
2. Define what must be true about the output (from completion criteria)
3. Write the evaluation criteria (how to judge pass/fail)
4. Include edge cases (missing data, ambiguous input, boundary conditions)

This is the test. Write it first. It will fail — there's no implementation yet. That's the point.

### 3. Red-Green-Refactor Cycle (Adapted)

The classic TDD cycle, adapted for functions that may include LLM calls:

| Phase | Traditional TDD | Governed Architecture TDD |
|-------|----------------|--------------------------|
| **Red** | Write a test that fails | Write golden examples with evaluation criteria — no implementation exists yet |
| **Green** | Write the minimum code to pass | Write the code + prompt + skill config that produces output meeting the criteria |
| **Refactor** | Clean up without breaking tests | Refine the prompt, adjust the skill config, optimise the code — golden examples must still pass |

**Key difference:** In traditional TDD, "Green" means the assertion passes exactly. In governed architecture TDD, "Green" means the evaluation criteria are satisfied — the output has the right structure, contains the required elements, and is faithful to the input.

### 4. Run, Evaluate, Iterate

For pure code functions: run the function, assert the output. Standard.

For code+LLM functions:
1. Run the function with the golden example input
2. Capture the output
3. Evaluate against the criteria (automated where possible, human review where necessary)
4. If criteria met → Green. If not → adjust code, prompt, or skill config → re-run.

For sub-agent functions: run each sub-agent independently with its golden input, evaluate each output separately, then test the merge step.

---

## Golden Example Structure

One golden example set per function. Each set contains multiple examples covering the happy path, edge cases, and failure modes.

### Example Entry

```markdown
# Golden Example: [Function Name]
# Step: [Step # from value chain map]
# Function Type: [pure code | code+LLM | sub-agent]
# Version: [N] | Last validated: [date]

## Example 1: [descriptive name — e.g., "complete input, clear match"]

### Input
[The concrete input data — real or realistic, sourced from stakeholder interviews or domain knowledge]

### Expected Output Characteristics
- [What must be TRUE about the output]
- [What must be PRESENT]
- [What must NOT be present]
- [Structural requirements — format, fields, completeness]

### Evaluation Criteria
| Criterion | Type | Required |
|-----------|------|----------|
| [e.g., "Every requirement addressed"] | completeness | yes |
| [e.g., "Evidence quoted from source, not fabricated"] | faithfulness | yes |
| [e.g., "Output is valid JSON matching schema"] | structure | yes |
| [e.g., "Confidence scores within 0-1 range"] | validity | yes |
| [e.g., "Processing completes within 10 seconds"] | performance | no |

### Edge Cases
| Variant | Input Modification | Expected Behaviour |
|---------|-------------------|-------------------|
| Missing data | [remove a key field] | [graceful handling — report gap, don't fabricate] |
| Ambiguous input | [add conflicting information] | [flag ambiguity, don't silently pick one] |
| Empty input | [provide nothing] | [clear error or "insufficient data" response] |
| Oversized input | [exceed expected volume] | [process or truncate with notice — don't crash] |
```

---

## Evaluation Criteria Types

When testing LLM-involving functions, evaluation criteria fall into these categories. Use the minimum set needed — not every function needs all types.

| Type | What It Checks | How to Evaluate |
|------|---------------|-----------------|
| **Completeness** | All required elements present in output | Checklist: is each expected element present? |
| **Faithfulness** | Output reflects the input accurately — no fabrication | Compare output claims against input source material |
| **Structure** | Output matches the expected format/schema | Schema validation (automated) |
| **Validity** | Values are within expected ranges and types | Range checks, type checks (automated) |
| **Relevance** | Output addresses the step's intention, no tangents | Human review or secondary LLM evaluation |
| **Consistency** | Same input produces compatible outputs across runs | Run N times, compare structural similarity |
| **Absence** | Forbidden content is NOT in the output | Negative checks: no hallucination, no PII leakage, no off-topic content |

✅ "Every requirement from the job description has a corresponding evidence entry, each evidence entry quotes text found in the CV" — completeness + faithfulness, verifiable.

❌ "The output should be good and accurate" — vague, untestable. If you can't define what "good" means in concrete criteria, you haven't defined the function.

---

## Artifacts

### 1. Golden Example Set

One per function. Contains all test cases for that function. Format as shown above.

```markdown
# Golden Example Set: [Function Name]
# Step: [#] | Type: [pure code | code+LLM | sub-agent]
# Examples: [count] | Edge cases: [count] | Last validated: [date]

## Example 1: [happy path]
[input, expected output characteristics, evaluation criteria]

## Example 2: [edge case — missing data]
[input variant, expected behaviour]

## Example 3: [edge case — ambiguous input]
[input variant, expected behaviour]

## Example N: [failure mode]
[input that should produce a graceful error or explicit "cannot process"]
```

### 2. Test Specification

One per step. Links the step to its golden examples and defines how to run the tests.

```markdown
# Test Specification: [Step Name]
# Step: [#] | Function: [name] | Type: [pure code | code+LLM | sub-agent]
# Version: [N]

## What Is Being Tested
[One sentence — the function's intention from the step specification]

## Golden Examples
[Link or reference to the golden example set]

## Pass Criteria
- All "required" evaluation criteria must be met
- [Threshold, if applicable — e.g., "8 of 10 requirements matched"]
- Edge cases must produce graceful handling, not crashes or fabrication

## How to Run
- [Isolated — no dependency on other steps running first]
- [Input: provided directly from golden example, not from live data]
- [Output: captured and stored for evaluation]

## Dependencies
- [Skill config required: name and version]
- [Prompt template required: reference]
- [Test data: source of golden example inputs]
```

### 3. Evaluation Rubric

For code+LLM functions where automated evaluation isn't sufficient. Defines how a human (or a secondary evaluation) judges the output.

```markdown
# Evaluation Rubric: [Function Name]
# Version: [N]

| Criterion | Weight | Pass | Fail |
|-----------|--------|------|------|
| [criterion] | [required/optional] | [what pass looks like] | [what fail looks like] |
```

---

## The Testing Hierarchy

Different function types need different testing approaches. Match the approach to the type.

| Function Type | Test Approach | Evaluation | Deterministic? |
|--------------|--------------|------------|----------------|
| **Pure code** | Standard unit tests — exact input/output assertions | Automated — assertEqual, schema validation | Yes |
| **Code + LLM** | Golden examples with evaluation criteria | Criteria-based — completeness, faithfulness, structure | No — criteria are deterministic, output varies |
| **Sub-agents** | Test each agent independently, then test the merge | Per-agent evaluation + merge correctness | Per-agent: No. Merge: Yes |

### Testing Sub-Agents

When a step uses parallel sub-agents (each with its own skill and dataset):

1. **Test each sub-agent in isolation** — its own golden example set, its own evaluation criteria
2. **Test the merge function separately** — given known sub-agent outputs, does the merge produce the correct combined result?
3. **Never test sub-agents through the merge** — if the merged output is wrong, you can't tell which agent failed

✅ Three sub-agents tested independently, merge tested with known inputs. Any failure points to exactly one component.

❌ One end-to-end test that runs all sub-agents and checks the final output. Failure gives you no information about which component broke.

---

## Anti-Patterns

| Mistake | Why It Fails | Fix |
|---------|-------------|-----|
| Writing code before golden examples | You don't know what "correct" looks like yet — you'll test against your assumptions, not against requirements | Golden examples first — derived from step specifications, not from implementation |
| Exact string matching on LLM output | LLM output varies across runs — tests become flaky and meaningless | Use evaluation criteria: structure, completeness, faithfulness — not exact text |
| Testing multiple steps together | When it fails, you can't isolate the cause — which step broke? | One golden example set per function, tested independently |
| Golden examples without edge cases | Happy path works, but missing data crashes the function in production | Every golden example set includes: missing data, ambiguous input, empty input |
| Vague evaluation criteria | "Output should be good" — untestable, undebugable | Every criterion must be binary: met or not met. If you can't judge it, it's not a criterion |
| Testing only once, then never again | Prompt changes, skill config changes, model upgrades — tests go stale | Run golden examples after every change to code, prompt, or skill config |
| Fabricating golden example inputs | Fake inputs don't expose real-world edge cases | Use real or realistic inputs from stakeholder interviews and value chain mapping |
| Skipping sub-agent isolation | Merged output hides individual failures | Test each sub-agent independently, test merge separately |

---

## Validation Checklist

Before considering a function tested:

- [ ] Golden examples derived from step specification (intention, I/O, completion criteria)?
- [ ] At least one happy-path example with full evaluation criteria?
- [ ] Edge cases covered: missing data, ambiguous input, empty input?
- [ ] Evaluation criteria are concrete and binary (met/not met)?
- [ ] For code+LLM: faithfulness criterion included (no fabrication)?
- [ ] For sub-agents: each tested independently + merge tested separately?
- [ ] Tests are isolated — no dependency on other steps or live data?
- [ ] Tests run after every change to code, prompt, or skill config?
- [ ] Golden example inputs sourced from real or realistic data?
- [ ] Zero vague criteria ("should be good", "should be accurate")?

---

## Artifact Governance

When creating artifacts from this skill:

1. Use the templates in `.claude/skills/artifact-template.md` — copy the relevant section (golden-example-set, test-spec, or eval-rubric)
2. Fill in the YAML frontmatter with project, version, status, and source
3. Register the artifact in `DESIGN.md` (project root) — add a row to the Artifact Registry table

---

## Referenced Skills

| Skill | Relationship |
|-------|-------------|
| **Value Chain Mapping** | Upstream — provides the step specifications that golden examples are derived from |
| **Data Evolution** | Peer — fields that are transformed need golden examples for the transformation |
| **Governed Architecture** | Peer — defines the function types (pure code, code+LLM, sub-agents) and their contracts |
| **Configure Claude Code** | Foundation — the development environment where tests are implemented and run |

---

*Write the test first. If you can't test it, you haven't defined it. Golden examples are the contract between intention and implementation.*
