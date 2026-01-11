# Claude Code Commands

Custom slash commands for Claude Code.

## Commands

### `/worktree [branch-name] [--stash] [--from <worktree>]`

Create a git worktree with synced configs, content migration, and background dependency installation.

**Usage:**

```bash
/worktree                           # Interactive branch selection
/worktree feature/new-api           # Specify branch name
/worktree feature/new-api --stash   # Migrate current changes to new worktree
/worktree feature/new-api --from main  # Migrate changes from another worktree
```

**Options:**

| Option | Description |
|--------|-------------|
| `--stash` | Save current uncommitted changes and apply them to the new worktree |
| `--from <name>` | Migrate uncommitted changes from the specified worktree |

**What it does:**

1. **Argument Parsing** - Extracts branch name and migration options
2. **Context Analysis** - Detects main repo vs existing worktree
3. **Branch Resolution** - Uses provided branch or shows recent branches
4. **Path Determination** - Uses unified `.worktrees/<project>/` structure
5. **Content Migration** - Stashes and applies changes if requested
6. **Worktree Creation** - Creates worktree in organized directory
7. **Config Sync** - Copies `.claude/`, `.env`, `.vscode/`, etc.
8. **Dependency Install** - Runs package manager in background
9. **Clipboard Copy** - Copies launch command to clipboard
10. **Summary Output** - Shows path and quick start instructions

**Directory Structure:**

```
parent/
├── project/                    # Main repo
└── .worktrees/
    └── project/                # Grouped by project name
        ├── feature-auth/       # Worktree 1
        ├── hotfix-123/         # Worktree 2
        └── ...
```

**Content Migration Examples:**

```bash
# Scenario: You have uncommitted changes and need to switch context
/worktree hotfix-urgent --stash
# → Stashes your current changes
# → Creates new worktree
# → Applies changes to new worktree
# → Original stash preserved for safety

# Scenario: Pull changes from another worktree
/worktree feature-b --from feature-a
# → Stashes changes from feature-a worktree
# → Creates new worktree feature-b
# → Applies those changes to feature-b
```

**Quick switch to new worktree:**

After `/worktree` completes, the command `cd <path> && claude` is copied to your clipboard.
Just press `Ctrl+C` to exit, then paste and run to start Claude Code in the new worktree.

**Synced files:**

- `.claude/` - Claude Code settings and commands
- `.env` / `.env.local` - Environment variables
- `.vscode/` - VS Code settings
- `.cursorrules` / `.windsurfrules` - Editor rules
- `docs/.local/` - Local documentation

## Installation

Copy `.claude/commands/` to your project root.

## References

- [Git Worktree Documentation](https://github.com/git/git/blob/master/Documentation/git-worktree.txt)
- [Claude Code Slash Commands](https://docs.anthropic.com/en/docs/claude-code)

## License

MIT
