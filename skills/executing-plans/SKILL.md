---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute all tasks, report when complete.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** Tell your human partner that Superpowers works much better with access to subagents. The quality of its work will be significantly higher if run on a platform with subagent support (such as Claude Code or Codex). If subagents are available, use superpowers:subagent-driven-development instead of this skill.

## The Process

### Step 1: Load and Review Plan
1. Read plan file
2. Review critically - identify any questions or concerns about the plan
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Create TodoWrite and proceed

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
  2. Run a read-only cross-harness review of the completed implementation.
     Use `review-code current implementation against <plan-file>`.
     - If running in Codex, use OpenCode with DeepSeek and the default OpenCode agent (do not pass `--agent`):
       ```bash
       OPENCODE_PERMISSION='{"edit":"deny","task":"deny","bash":{"*":"deny","git diff*":"allow","git log*":"allow","git status*":"allow","git show*":"allow","rg *":"allow","grep *":"allow","sed *":"allow"}}' \
       opencode run "Use my skills. Use the review-code skill. Arguments: current implementation against <plan-file>. Return only the review." --model deepseek/deepseek-v4-flash --variant high --dir <repo>
       ```
     - If running in OpenCode, ask it to run Codex with GPT-5.5 and medium reasoning effort in read-only mode:
       ```text
       Can you run `codex exec --model gpt-5.5 -c model_reasoning_effort="medium" -a never --sandbox read-only -C <repo> "Use the review-code skill. Arguments: current implementation against <plan-file>"`?
       ```
     - If the opposite harness or target model is unavailable, fall back to `/review` and note the fallback.
  3. Run the review loop at most 6 review/address iterations. Each iteration is:
     run the review command, read the returned review, address the review, then
     re-run the review command if needed. Stop on `Verdict: Approve` or no
     Required/Concern improvements; otherwise fix valid Required/Concern
     feedback and explain rejected feedback before the next review. If the base
     implementer disagrees with all Required/Concern suggestions in an
     iteration, stop the loop and record the disagreement. Do not run reviews
     back-to-back without addressing findings, and do not keep looping for Nits
     only.
  4. Output a commit message for the implemented changes.

The reviewer must not edit files, run lint, run tests, install dependencies, or
perform cleanup. Code ready for review must already have been linted/tested by
the implementer according to the plan's verification steps.

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
- **superpowers:writing-plans** - Creates the plan this skill executes
- **superpowers:finishing-a-development-branch** - Complete development after all tasks
