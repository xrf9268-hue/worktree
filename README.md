# Claude Code Commands

Custom slash commands for Claude Code.

## Commands

### `/worktree [branch-name]`

Create a git worktree with synced configs and background dependency installation.

**Usage:**

```bash
/worktree                    # Interactive branch selection
/worktree feature/new-api    # Specify branch name
```

**What it does:**

1. **Context Analysis** - Detects main repo vs existing worktree
2. **Branch Resolution** - Uses provided branch or shows recent branches for selection
3. **Worktree Creation** - Creates worktree as sibling directory
4. **Config Sync** - Copies `.claude/`, `.env`, `.vscode/`, etc.
5. **Dependency Install** - Runs package manager in background
6. **Summary Output** - Shows path and next steps

**Synced files:**

- `.claude/` - Claude Code settings
- `.env` / `.env.local` - Environment variables
- `.vscode/` - VS Code settings
- `.cursorrules` / `.windsurfrules` - Editor rules
- `docs/.local/` - Local documentation

## Installation

Copy `.claude/commands/` to your project root.

## License

MIT
