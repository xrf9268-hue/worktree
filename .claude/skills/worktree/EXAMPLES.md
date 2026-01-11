# Usage Examples

## Basic Usage

### Create worktree for existing branch

```
/worktree feature-auth
```

Creates a worktree at `../.worktrees/<project>/feature-auth/`

### Create worktree for new branch

```
/worktree my-new-feature
```

If branch doesn't exist, creates it with `-b` flag.

### Interactive branch selection

```
/worktree
```

Shows recent branches and prompts for selection.

---

## With Content Migration

### Move current changes to new worktree

```
/worktree feature-auth --stash
```

1. Stashes current uncommitted changes
2. Creates worktree
3. Applies stash in new worktree

### Move changes from another worktree

```
/worktree feature-auth --from hotfix-123
```

1. Stashes changes from `hotfix-123` worktree
2. Creates `feature-auth` worktree
3. Applies stash in new worktree

---

## Natural Language Triggers

The skill also responds to natural language:

- "Create a parallel branch for feature work"
- "I need a new worktree for testing"
- "Switch to a new branch but keep my changes"
- "Set up a parallel development environment"

---

## Output Example

```
Worktree created successfully!

  Path:     ../.worktrees/my-project/feature-auth/
  Branch:   feature-auth
  Configs:  .claude/, .env, .vscode/

  Migrated: Changes applied from current worktree
            Original stash preserved for safety

  Dependencies: Installing in background (pnpm)

Quick start (copied to clipboard):
  1. Press Ctrl+C to exit current session
  2. Paste and run: cd ../.worktrees/my-project/feature-auth && claude
```
