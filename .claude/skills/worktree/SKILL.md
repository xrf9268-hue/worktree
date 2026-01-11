---
name: worktree
description: Create a git worktree with synced configs, content migration, and background dependency installation. Use when creating parallel development environments, switching branches with uncommitted changes, or when user mentions "worktree", "parallel branch", "new environment", or "switch branch with changes".
argument-hint: [branch-name] [--stash] [--from <worktree>]
allowed-tools:
  - Bash(git:*)
  - Bash(cp:*)
  - Bash(mkdir:*)
  - Bash(ls:*)
  - Bash(cat:*)
  - Bash(cd:*)
  - Bash(pnpm:*)
  - Bash(npm:*)
  - Bash(yarn:*)
  - Bash(bun:*)
  - Bash(echo:*)
  - Bash(command:*)
  - Bash(pbcopy:*)
  - Bash(xclip:*)
  - Bash(xsel:*)
  - Bash(wl-copy:*)
  - Bash(date:*)
  - Bash(grep:*)
  - Bash(awk:*)
  - Bash(pwd:*)
  - Bash(tr:*)
user-invocable: true
---

# Git Worktree Workflow

You are an expert DevOps assistant. Create a parallel development environment with optional content migration.

## Quick Start

1. **Parse arguments** from `$ARGUMENTS`: branch name, `--stash`, `--from <name>`
2. **Analyze context**: Find main repo via `git rev-parse --git-common-dir`
3. **Resolve branch**: Use provided name or prompt user to select
4. **Create worktree** at `../.worktrees/<project>/<branch>/`
5. **Sync configs**: Copy `.claude/`, `.env`, `.vscode/`, etc.
6. **Install deps**: Background install with detected package manager
7. **Copy launch command** to clipboard

## Arguments

| Argument | Description |
|----------|-------------|
| `branch-name` | Target branch (positional) |
| `--stash` | Migrate current uncommitted changes to new worktree |
| `--from <name>` | Migrate changes from specified worktree |

## Directory Structure

```
parent/
├── project/                    # Main repo
└── .worktrees/
    └── project/                # Grouped by project
        ├── feature-auth/       # Worktree 1
        └── hotfix-123/         # Worktree 2
```

## Detailed Documentation

- Complete workflow steps: [WORKFLOW.md](WORKFLOW.md)
- Content migration guide: [MIGRATION.md](MIGRATION.md)
- Usage examples: [EXAMPLES.md](EXAMPLES.md)

## Output

Provide a clear summary (do NOT auto-open editor):

```
Worktree created successfully!

  Path:     ../.worktrees/<project>/<branch>/
  Branch:   <branch-name>
  Configs:  .claude/, .env, .vscode/, ...
  Migrated: [if applicable]
  Dependencies: Installing in background (<pkg-mgr>)

Quick start (copied to clipboard):
  cd <path> && claude
```
