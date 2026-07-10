---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge or cleanup
---

# Finishing a Development Branch

## Overview

Guide completion of development work by presenting clear options and handling chosen workflow.

**Core principle:** Verify tests → Detect environment → Present options → Execute choice → Clean up.

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."

## The Process

### Step 1: Verify Tests

**Before presenting options, satisfy `verification-before-completion`** — run the project's
verifying command this turn and confirm it passes. That skill owns *how* to verify; don't restate
test commands here.

**If verification fails:**
```
Verification failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with integration until it passes.
```

Stop. Don't proceed to Step 2.

**If it passes:** Continue to Step 2.

### Step 2: Detect Environment

Two facts decide the menu and cleanup:

```bash
branch=$(git branch --show-current)          # empty ⇒ detached HEAD
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

- **Ours to integrate** — on a named `branch` **and** the workspace is one we can clean up: a normal
  repo (`GIT_DIR == GIT_COMMON`) or a worktree under `.worktrees/` or `worktrees/`.
- **Hand-off** — anything else: a detached HEAD (empty `branch`), or a worktree the harness owns (any
  other path). This skill does **not** integrate or clean up a hand-off workspace — the harness owns
  its lifecycle.

| State | Menu | Cleanup |
|-------|------|---------|
| Named branch + normal repo or our worktree | Integrate menu (4 options) | Options 2 & 4 clean up; Option 1 self-cleans |
| Detached HEAD, or harness-owned worktree | Hand-off menu (3 options) | None — the harness owns the workspace |

### Step 3: Determine Base Branch

Resolve the base **branch name** (`main` or `master`) — not a commit — because the menu and
`squash-merge` both take the literal name:

```bash
git show-ref --verify --quiet refs/heads/main   && has_main=1
git show-ref --verify --quiet refs/heads/master && has_master=1
```

Use whichever single one exists. If **both** exist, ask the user to pick `main` or `master` (as a
structured single-select; `squash-merge` refuses to guess). If **neither** exists, stop —
`squash-merge` requires an existing local `main` or `master`; report the unsupported state instead of
rendering the menu.

### Step 4: Present Options

Ask the choice as a **structured single-select question** — one option per item below, each with a
one-line preview, per the "Asking structured questions" rule. Fall back to printing the text block
verbatim only in a harness with no structured-question tool. Keep the exact option set and order.

**Ours to integrate — 4 options:**

```
1. Squash-merge into <base-branch> via bin/squash-merge
2. Merge into <base-branch> without squashing (preserve commits)
3. Keep the branch as-is (I'll handle it later)
4. Discard this work
```

**Hand-off (detached HEAD or harness-owned worktree) — 3 options:**

```
1. Name this checkout as a branch and hand off to the harness
2. Keep as-is (I'll handle it later)
3. Discard this work
```

**Don't add explanation** - keep options concise.

### Step 5: Execute Choice

#### Integrate — Option 1: Squash-merge (default)

Run from inside the feature worktree/branch. `squash-merge` squash-merges the branch into the base,
generates the Conventional Commit message, commits, then removes the feature worktree and deletes
the branch — it handles its own cleanup, so **skip Step 6 for this option.**

```bash
squash-merge <base-branch>
```

The script integrates and deletes the branch in one shot, so verify the base branch afterward (per
`verification-before-completion`) and fix forward if it fails — the branch is already gone.

#### Integrate — Option 2: Merge without squashing

Preserves the individual commits as a merge into the base.

```bash
# Get main repo root for CWD safety
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"

# Merge first — verify success before removing anything
git checkout <base-branch>
git merge <feature-branch>
```

Verify the merged base (per `verification-before-completion`) before removing anything. Then:
Cleanup worktree (Step 6), then delete branch (you're on `<base-branch>` now, so the delete
succeeds):

```bash
git branch -d <feature-branch>
```

#### Integrate — Option 3: Keep As-Is

Report: "Keeping branch <name>. Worktree preserved at <path>."

**Don't cleanup worktree.**

#### Integrate — Option 4: Discard

**Confirm first** (free-form typed confirmation, not a menu pick — exact text matters):
```
This will permanently delete:
- Branch <name>
- All commits: <commit-list>
- Worktree at <path>

Type 'discard' to confirm.
```

Wait for exact confirmation. If confirmed:
```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
```

Then: Cleanup worktree (Step 6) — that frees the branch in a worktree setup. In a **normal repo**
there's no worktree, so you're still on the feature branch; switch off it before deleting
(`git branch -D` refuses the current branch):

```bash
git checkout <base-branch>   # normal repo only; skip if Step 6 already removed the worktree
git branch -D <feature-branch>
```

#### Hand-off execution (detached HEAD or harness-owned worktree)

The harness owns this workspace — **never `git worktree remove` it, and don't run `squash-merge`**
(it removes the worktree). Step 6 does not apply.

- **Hand-off 1 (name + hand off):** if detached, `git switch -c <feature-branch>` to name the work;
  report the branch name so the harness can integrate it. Change nothing else.
- **Hand-off 2 (keep as-is):** report the current state; change nothing.
- **Hand-off 3 (discard):** after typed `discard` confirmation, delete only a branch you created here
  (a fresh detached HEAD has none until you name one); otherwise report that the harness must discard
  the workspace. Leave the workspace in place.

### Step 6: Cleanup Workspace

**Only runs for integrate Options 2 and 4.** Option 1 (squash-merge) self-cleans; Option 3 (keep)
and every hand-off option preserve the workspace.

Step 2 already routed harness-owned workspaces to hand-off, so here the worktree is one we own:

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
# A normal repo has no worktree to remove — skip this line there.
git worktree remove "$WORKTREE_PATH"
git worktree prune  # Self-healing: clean up any stale registrations
```

## Quick Reference

| Option | Integrate | Keep Worktree | Cleanup Branch |
|--------|-----------|---------------|----------------|
| 1. Squash-merge (script) | squash | - | yes (via script) |
| 2. Merge without squashing | merge commit | - | yes |
| 3. Keep as-is | - | yes | - |
| 4. Discard | - | - | yes (force) |
| Hand-off 1/2/3 | harness | yes | - |

## Common Mistakes

**Skipping test verification**
- **Problem:** Merge broken code
- **Fix:** Always satisfy `verification-before-completion` before offering options

**Open-ended questions**
- **Problem:** "What should I do next?" is ambiguous
- **Fix:** Present the exact structured option set (4 integrate, or 3 hand-off)

**Integrating a harness-owned or detached workspace**
- **Problem:** Running `squash-merge`/`git worktree remove` on a workspace the harness owns corrupts its state; branch commands fail on a detached HEAD
- **Fix:** Route those to the hand-off menu — name a branch and let the harness integrate

**Deleting branch before removing worktree**
- **Problem:** `git branch -d` fails because a worktree still references the branch
- **Fix:** Merge/remove the worktree first, then delete the branch

**Running git worktree remove from inside the worktree**
- **Problem:** Command fails silently when CWD is inside the worktree being removed
- **Fix:** Always `cd` to main repo root before `git worktree remove`

**No confirmation for discard**
- **Problem:** Accidentally delete work
- **Fix:** Require typed "discard" confirmation (free-form, not a menu pick)

## Red Flags

**Never:**
- Proceed with failing verification
- Merge without verifying tests on the result
- Delete unmerged or discarded work without confirmation
- Remove a worktree before confirming merge success
- Remove or integrate a harness-owned / detached workspace (hand it off instead)
- Run `git worktree remove` from inside the worktree

**Always:**
- Verify tests before offering options
- Detect environment before presenting menu
- Present the exact structured option set (4 integrate / 3 hand-off)
- Get typed confirmation for discard
- Clean up worktree for integrate Options 2 & 4 only (Option 1 self-cleans; hand-off never cleans)
- `cd` to main repo root before worktree removal
- Run `git worktree prune` after removal
