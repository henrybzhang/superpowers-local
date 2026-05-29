# Code Quality Reviewer Prompt Template

Use this template when running a code quality reviewer.

**Purpose:** Verify implementation is well-built (clean, tested, maintainable)

**Only dispatch after spec compliance review passes.**

**Review-only:** You may run targeted, non-destructive checks, but do not edit
files, install dependencies, update snapshots, regenerate committed artifacts,
run migrations against real services, start long-lived processes, or perform
cleanup.

Use this as a targeted per-task review prompt, not as a full implementation
review. The full four-lane code review runs only after all tasks are complete.

## What Was Requested

[FULL TEXT of task requirements]

## What Implementer Claims They Built

[From implementer's report]

## Changed Code

Review the task diff, usually `<commit-before-task>..HEAD`, and inspect the
changed files plus nearby context needed to judge this task.

## Your Job

Verify the task implementation is well-built:

- Does each changed file have a clear responsibility?
- Are changed units decomposed so they can be understood and tested?
- Does the implementation follow the file structure and ownership in the plan?
- Are tests or validation evidence meaningful for this task?
- Did this task introduce avoidable duplication, unnecessary abstraction, or
  needless complexity?
- Did this task create new files that are already large, or significantly grow
  existing files in a way that makes the task harder to understand?

**Code reviewer returns:** `Verdict: Approve | Revise` with Required, Concern, or Nit findings.
