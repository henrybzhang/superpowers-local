---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** Planning and implementation should happen on a separate branch in a dedicated worktree. Use `superpowers:using-git-worktrees` before writing or executing plans unless the user explicitly says not to. Follow repository instructions for worktree location.

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)
- Keep the plan as a local working artifact: available for execution, left untracked, and excluded from commits

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Code Block Discipline

Inlining full implementation bodies is the most common bloat failure. A plan describes intent and contracts; it is not a working draft of the code itself. Subagents and skilled humans both write better code from a tight contract than from a verbatim recipe.

**Reproducibility test:** Given the signature, types, data tables, and a 2–4 line algorithm summary, could a skilled dev reproduce a working implementation? If yes → omit the body. If no → keep it.

## Contract Precision Without Boilerplate

Subagents and future plans may depend on exact interfaces from earlier tasks. Plans MUST pin these contract details wherever introduced:
- Exact function, method, class, helper, constant, and file names.
- Argument names and types, return types, and exported/internal visibility expectations.
- Mutated fields, emitted events, persistence effects, and return-value shapes.
- Formulas, thresholds, clamp ranges, enum/string values, and ordering rules.
- Test names when later tasks or verification steps reference them.

Do not preserve repeated setup boilerplate just to be explicit. If a test body is mostly fixture construction, repeated model initialization, or obvious arrange code:
- Define or name a shared helper once with its exact signature and required field values.
- For each test, list the variant inputs, action, and assertions.
- Keep the full body only when the setup itself is the contract or would be hard to reconstruct safely.

**Always include in full:**
- **Contract tests** — tests whose assertion *values* are the design: parser input→output cases, scheduler placement under specific calendar state, off-by-one edge cases, computational behavior with non-obvious mappings, and anything in a task whose function/hook/service is imported by a later task. These act as executable inter-task contracts — a subagent implementing a downstream task can rely on the asserted values without reading the upstream implementation. When in doubt, keep in full.
- Type definitions, function signatures, API shapes.
- Data tables and constants — priority maps, terrain rules, copy strings, magic numbers, TTLs, thresholds. These ARE the design.
- Algorithms with off-by-one risk, undocumented invariants, or performance constraints (e.g. Bresenham walks, custom hashing, tight loops).
- Irreversible ops — migrations, fs writes, network mutations.

**Omit (replace with an outline block):**
- Dispatch on union variants where each branch is a small obvious mutation.
- Simple filter / map / reduce / sort over already-typed data.
- Plumbing that calls already-specified helpers in sequence.
- Getters, setters, thin wrappers, aliases.
- Narration code — logs, telemetry events, push-event calls.
- Structural UI tests — render-and-assert tests where the assertions are DOM presence, ARIA role, callback firing, or simple classname checks, AND the component is rendered only within the current task. Compress to a one-line behavior bullet so the implementer writes the RTL boilerplate. Keep in full whenever the component is reused in a later task or has non-obvious prop/state combinations.

**Outline block template** — use this in place of an omitted body:

```markdown
**`functionName(args): ReturnType` — outline:**
- Algorithm: <2–4 bullets, one per logical step>
- Data tables (if any): <inline the table>
- Edge cases: <only the non-obvious ones>
- Side effects: <which fields on which objects, with magnitudes>
```

Magnitudes, field names, and table contents ARE the design — keep them. The boilerplate around them is not.

**Test outline template** — for structural tests, use one bullet per behavior:

```markdown
- Test: <one-line behavior, naming the input event and the expected effect>
  - e.g. "Test: clicking the row fires `onSelect` with the row's id"
  - e.g. "Test: shows count badge only when count > 0"
  - e.g. "Test: pressing Esc closes the slide-over"
```

The behavior line IS the spec. The implementer writes the test setup, render wrapper, and assertions.

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

**Branch/Worktree:** Implement this plan on a separate branch in a dedicated worktree. Follow repository instructions for the worktree directory.

---
```

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

Commit after this task.
- Files: `tests/path/test.py`, `src/path/file.py`
- Suggested subject: `feat: add specific feature`
- Verification: `pytest tests/path/test.py::test_name -v`
Follow the repository instruction file for commit format and body requirements.
````

## No Placeholders

Every step must contain the actual content an engineer needs. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" — pin the contract in the task that introduces it (types, signatures, schemas); reference it by name in later tasks
- Steps that describe what to do without showing how — see Code Block Discipline for the full keep/omit rules (tests, types, signatures, data tables, non-trivial algorithms, irreversible ops) vs. what should be an outline block instead
- References to types, functions, or methods not defined in any task

## Remember
- Exact file paths always
- Exact interfaces and contract values always; compress repeated setup boilerplate with named helpers and per-test behavior bullets
- Tests, types, signatures, data tables in full when they define a contract; implementation bodies omitted unless they meet the keep-list in Code Block Discipline — use an outline block otherwise
- Exact commands with expected output
- DRY, YAGNI, TDD, frequent commits; reference repository instruction files for detailed commit-message rules instead of repeating full commit commands

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a task that implements it? List any gaps.

**2. Placeholder scan:** Search your plan for red flags — any of the patterns from the "No Placeholders" section above. Fix them.

**3. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

**4. Body audit:** Two passes.

*Implementation bodies:* for every code block over ~15 lines that isn't a test, type definition, or data table — does it pass the reproducibility test in Code Block Discipline? If yes, replace with an outline block. Magnitudes and table contents stay; dispatch boilerplate goes.

*Test bodies:* for every test block, classify it.
- **Contract test** (values are the design; consumed by another task) → keep in full.
- **Structural test** (presence/role/callback/classname; single-task scope) → compress to a behavior bullet using the test outline template.
- **Fixture-heavy contract test** (assertions are important, setup is repeated boilerplate) → define a helper once with exact signature/required fields, then list each test's inputs/action/assertions.

If you can't tell which a test is, ask: "would removing this test's body force a downstream subagent to read this task's source code to know how to call the function?" If yes → contract; keep. If no → structural; compress.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no task, add the task.

## Cross-Harness Plan Review

After self-review passes, run a read-only external plan review with the
`review-plan` skill from `../custom-commands/review-plan`.

**If running in Codex:** run the OpenCode wrapper outside the Codex sandbox.

```bash
opencode-review-plan <plan-file> <spec-file>
```

**If running in OpenCode:** run the Codex wrapper.

```bash
codex-review-plan <plan-file> <spec-file>
```

The reviewer may run targeted, non-destructive checks as allowed by the review
skill. It must not edit files, install dependencies, update snapshots,
regenerate committed artifacts, run migrations against real services, start
long-lived processes, or perform cleanup. If the opposite harness or target
model is unavailable, run the review in the current harness and note the
fallback before offering execution.

**Review loop:** Follow `workflow-policy` for interactive sessions:
`review-plan`, at most 3 review/address iterations, stop on `Verdict: Approve`
or when no accepted Required/Concern improvements remain. Apply valid feedback,
explain rejected feedback briefly, never run reviews back-to-back without
addressing findings, and do not keep looping for Nits only.

## Execution Handoff

After saving the plan, offer execution choice:

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?"**

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- Fresh subagent per task + two-stage review

**If Inline Execution chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans
- Batch execution with checkpoints for review
