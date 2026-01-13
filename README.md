# beads `bd doctor --fix` Sync Branch Bug Reproduction

This repository demonstrates a bug where `bd doctor --fix` destroys the sync branch history by resetting it to main.

## The Bug

When using beads with a sync-branch workflow, `bd doctor` incorrectly warns that the sync branch is "behind main on source files" and offers to "fix" it by resetting the sync branch to main and force pushing. This destroys the sync branch history.

The sync branch is **supposed** to diverge from main - it only tracks `.beads/` data, not source code.

## Reproduction Steps

**Note:** Fork this repo first - you'll need push access to see the full destructive behavior.

```bash
# 1. Fork this repo on GitHub, then clone your fork
git clone https://github.com/YOUR_USERNAME/beads-doctor-fix-bug-repro.git
cd beads-doctor-fix-bug-repro

# 2. Verify the starting state
git log --oneline beads-sync -3
# Should show: e66be4c Initialize beads (the correct state)

# 3. Run bd doctor and observe the warning
bd doctor
# You'll see:
#   ⚠  Sync Branch Health: Sync branch 171 commits behind main on source files
#      └─ 51 source files differ between beads-sync and main. The sync branch has stale code.

# 4. Run the "fix" (THIS DESTROYS THE SYNC BRANCH)
bd doctor --fix --yes

# 5. Observe the damage
git log --oneline beads-sync -3
# Now shows main's commits (file50.txt, file49.txt, etc.) instead of beads commits
```

## Resetting After Reproduction

Tags are provided to reset the repo to its starting state:

```bash
# Reset beads-sync to correct state (via worktree)
git -C .git/beads-worktrees/beads-sync reset --hard repro-beads-sync-correct
git push --force origin beads-sync

# Verify
git log --oneline beads-sync -3
# Should show: e66be4c Initialize beads
```

## Tags

- `repro-main-head` - Where main should be for reproduction
- `repro-beads-sync-correct` - Where beads-sync should be (the correct state that gets destroyed)

## Expected Behavior

The "Sync Branch Health" check should NOT warn about source file differences between the sync branch and main. This divergence is expected and correct behavior for a sync-branch workflow.

`bd doctor --fix` should NEVER reset a sync branch to main.

## Actual Behavior

```
Fixing Sync Branch Health...
  Resetting sync branch in worktree: .git/beads-worktrees/beads-sync
  ✓ Reset beads-sync to main and pushed
  ✓ Fixed
```

This destroys all sync history and force-pushes to remote, affecting all collaborators.
