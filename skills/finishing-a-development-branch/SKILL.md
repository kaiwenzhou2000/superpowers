---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to integrate the work - automatically selects and executes the appropriate workflow (merge, PR, or local completion)
---

# Finishing a Development Branch

## Overview

Guide completion of development work by automatically selecting the appropriate workflow based on project context.

**Core principle:** Verify tests → Auto-select best option → Execute workflow → Clean up.

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."

## The Process

### Step 1: Verify Tests

**Before presenting options, verify tests pass:**

```bash
# Run project's test suite
npm test / cargo test / pytest / go test ./...
```

**If tests fail:**
```
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with merge/PR until tests pass.
```

Stop. Don't proceed to Step 2.

**If tests pass:** Continue to Step 2.

### Step 2: Determine Base Branch

```bash
# Try common base branches
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

Or ask: "This branch split from main - is that correct?"

### Step 3: Auto-Select Completion Option

Automatically determine the best completion option based on context:

**Decision Logic:**

1. **Check if branch is already pushed:**
   ```bash
   git branch -vv | grep $(git branch --show-current)
   ```

2. **Check if remote exists:**
   ```bash
   git remote -v | grep origin
   ```

3. **Auto-select based on context:**
   - If branch is already pushed → **Option 2: Create PR** (user expects to share work)
   - If remote exists and branch is not pushed → **Option 2: Create PR** (standard collaborative workflow)
   - If no remote exists → **Option 1: Merge locally** (local-only project)

**Notify user of auto-selected option:**
```
Implementation complete. Auto-selecting completion option based on context:

→ Creating Pull Request (branch is ready for collaborative review)

[or]

→ Merging locally to <base-branch> (local project, no remote configured)
```

**User can still choose differently:** If the user immediately responds with a preference (1-4), honor their choice instead.

### Step 4: Execute Choice

#### Option 1: Merge Locally

```bash
# Switch to base branch
git checkout <base-branch>

# Pull latest
git pull

# Merge feature branch
git merge <feature-branch>

# Verify tests on merged result
<test command>

# If tests pass
git branch -d <feature-branch>
```

Then: Cleanup worktree (Step 5)

#### Option 2: Push and Create PR

```bash
# Push branch
git push -u origin <feature-branch>

# Create PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

Then: Cleanup worktree (Step 5)

#### Option 3: Keep As-Is

Report: "Keeping branch <name>. Worktree preserved at <path>."

**Don't cleanup worktree.**

#### Option 4: Discard

**Confirm first:**
```
This will permanently delete:
- Branch <name>
- All commits: <commit-list>
- Worktree at <path>

Type 'discard' to confirm.
```

Wait for exact confirmation.

If confirmed:
```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

Then: Cleanup worktree (Step 5)

### Step 5: Cleanup Worktree

**For Options 1, 2, 4:**

Check if in worktree:
```bash
git worktree list | grep $(git branch --show-current)
```

If yes:
```bash
git worktree remove <worktree-path>
```

**For Option 3:** Keep worktree.

## Quick Reference

| Context | Auto-Selected Action | Merge | Push | Keep Worktree | Cleanup Branch |
|---------|---------------------|-------|------|---------------|----------------|
| Branch already pushed | Create PR (Option 2) | - | ✓ | ✓ | - |
| Remote exists, not pushed | Create PR (Option 2) | - | ✓ | ✓ | - |
| No remote configured | Merge locally (Option 1) | ✓ | - | - | ✓ |

**Manual Override:** User can still choose Options 3 (keep as-is) or 4 (discard) by responding immediately.

## Common Mistakes

**Skipping test verification**
- **Problem:** Merge broken code, create failing PR
- **Fix:** Always verify tests before auto-selecting option

**Wrong context detection**
- **Problem:** Choose merge when user expects PR, or vice versa
- **Fix:** Check both branch tracking and remote existence, notify user of choice

**Automatic worktree cleanup**
- **Problem:** Remove worktree when might need it (PR workflow)
- **Fix:** Only cleanup for merge locally workflow

**No confirmation for discard**
- **Problem:** Accidentally delete work
- **Fix:** Still require typed "discard" confirmation (safety preserved)

## Red Flags

**Never:**
- Proceed with failing tests
- Merge without verifying tests on result
- Delete work without confirmation (Option 4 still requires typed "discard")
- Force-push without explicit request
- Auto-select discard option (always requires user confirmation)

**Always:**
- Verify tests before auto-selecting option
- Detect project context (remote status, branch tracking)
- Notify user of auto-selected action with reasoning
- Allow immediate user override if they respond with preference
- Get typed confirmation for Option 4 (discard)

## Integration

**Called by:**
- **subagent-driven-development** (Step 7) - After all tasks complete
- **executing-plans** (Step 5) - After all batches complete

**Pairs with:**
- **using-git-worktrees** - Cleans up worktree created by that skill
