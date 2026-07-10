---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute all plan tasks, run mandatory
post-implementation gates, then complete the branch workflow. Work is not
complete until Step 4 finishes.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** Tell your human partner that Superpowers works much better with access to subagents. The quality of its work will be significantly higher if run on a platform with subagent support (such as Claude Code or Codex). If subagents are available, use superpowers:subagent-driven-development instead of this skill.

## The Process

## Definition of Complete

You are NOT done when implementation tasks pass.

You are done only after:
1. All plan tasks are completed and verified
2. The post-implementation checklist has completed, or a required
   fallback/blocker has been reported
3. superpowers:finishing-a-development-branch has run

If any post-implementation command cannot run, do not skip it silently. State
the exact blocker, use the documented fallback if available, and continue only
if the fallback satisfies the checklist.

### Step 1: Load and Review Plan
1. Read plan file
2. Review critically - identify any questions or concerns about the plan
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Create TodoWrite and proceed

TodoWrite MUST include:
- Each plan task
- Run `/simplify`
- Run verification after `/simplify`
- Run cross-harness `review-code`
- Adjudicate and revise accepted findings at every severity, including Nits and
  low-level findings
- Must adjudicate accepted findings at every severity before re-review
- Run verification after accepted review fixes
- Output commit message
- Run superpowers:finishing-a-development-branch

### Step 2: Execute Tasks

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. Mark as completed

### Step 3: Post-Implementation Checklist

After all tasks complete and verified:
- **REQUIRED:** Run the post-implementation checklist:
  1. Run the command `/simplify` to simplify the current changes.
  2. Re-run the plan's verification after simplification. Record the exact
     command output, or record the exact blocker if verification cannot run.
  3. Run a read-only cross-harness review of the completed implementation only
     after verification evidence exists or the blocker has been recorded. Use
     `review-code <target> [against <plan-or-requirements>]`; for the common
     plan-backed case, use `review-code current implementation against
     <plan-file>`.
     - If running in Codex, run the OpenCode wrapper outside the Codex sandbox:
       ```bash
       opencode-review-code <target> [against <plan-or-requirements>]
       ```
     - If running in OpenCode, run the Codex wrapper:
       ```bash
       codex-review-code <target> [against <plan-or-requirements>]
       ```
     - If the opposite harness or target model is unavailable, use
       `review-code` in the current harness/model and note the fallback.
  4. Follow `workflow-policy` for interactive implementation sessions:
     `review-code`, using the applicable review/address iteration cap, stop on
     `Verdict: Approve` or when no accepted improvements remain. Adjudicate all
     findings, including Nits and low-level findings. Apply valid feedback,
     explain rejected feedback briefly, run verification after accepted review
     fixes, and never run reviews back-to-back without addressing accepted
     findings.
  5. Output a commit message for the implemented changes.

Outputting the commit message is not the final response. Immediately continue
to Step 4 unless blocked.

review-code inspects existing evidence. It is not responsible for running
deterministic checks, lint, typecheck, build, tests, coverage, snapshots,
generated artifacts, services, migrations, or cleanup. Code ready for review
must already have been linted/tested by the implementer according to the plan's
verification steps, or the exact verification blocker must be recorded.

### Step 4: Complete Development

After the post-implementation checklist completes:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** - stop and ask.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Reference skills when plan says to
- Stop when blocked, don't guess
- Never start implementation on main/master branch without explicit user consent

## Integration

**Required workflow skills:**
- **superpowers:using-git-worktrees** - Ensures isolated workspace (creates one or verifies existing)
- **writing-plans** - Creates the plan this skill executes
- **superpowers:finishing-a-development-branch** - Complete development after all tasks
