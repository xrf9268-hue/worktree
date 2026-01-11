# Content Migration Guide

This document explains the two migration options for moving uncommitted changes to a new worktree.

## Option A: `--stash` Flag

Migrate **current** uncommitted changes to the new worktree.

### When to Use

- You have work-in-progress in the current directory
- You want to continue that work in a new branch/worktree

### How It Works

```bash
if [ "$USE_STASH" = true ]; then
  # Check for uncommitted changes
  if [ -n "$(git status --porcelain)" ]; then
    STASH_MSG="worktree-migrate-$(date +%Y%m%d-%H%M%S)"

    # Stash including untracked files
    git stash push -u -m "$STASH_MSG"
    STASH_CREATED=true

    echo "Changes stashed: $STASH_MSG"
  else
    echo "No changes to migrate"
    STASH_CREATED=false
  fi
fi
```

### Key Points

- Uses `git stash push -u` to include untracked files
- Creates a timestamped stash message for identification
- Original stash is preserved after applying (for safety)

---

## Option B: `--from <worktree>` Flag

Migrate changes from **another existing worktree**.

### When to Use

- You have work-in-progress in a different worktree
- You want to move that work to a new branch/worktree

### How It Works

```bash
if [ -n "$FROM_WORKTREE" ]; then
  # Find source worktree path
  SOURCE_PATH=$(git worktree list | grep "$FROM_WORKTREE" | awk '{print $1}')

  if [ -z "$SOURCE_PATH" ]; then
    echo "Error: Worktree '$FROM_WORKTREE' not found"
    git worktree list
    exit 1
  fi

  # Check for changes in source
  if [ -n "$(git -C "$SOURCE_PATH" status --porcelain)" ]; then
    STASH_MSG="migrate-from-${FROM_WORKTREE}-$(date +%s)"
    git -C "$SOURCE_PATH" stash push -u -m "$STASH_MSG"
    FROM_STASH_CREATED=true
    echo "Changes stashed from $FROM_WORKTREE: $STASH_MSG"
  else
    echo "No changes in source worktree"
    FROM_STASH_CREATED=false
  fi
fi
```

### Key Points

- Searches existing worktrees by name
- Shows error and lists worktrees if source not found
- Uses `git -C` to operate on remote worktree

---

## Applying Migrated Changes

After worktree creation, changes are applied automatically:

```bash
if [ "$STASH_CREATED" = true ] || [ "$FROM_STASH_CREATED" = true ]; then
  git -C "$NEW_PATH" stash apply "stash@{0}"
  echo "Changes applied to new worktree"
  echo "Original stash preserved (use 'git stash drop' to remove)"
fi
```

## Safety Notes

1. **Stash is preserved**: The original stash is NOT dropped automatically
2. **Manual cleanup**: Use `git stash drop` after confirming changes applied correctly
3. **Conflict handling**: If apply fails due to conflicts, resolve manually in new worktree
