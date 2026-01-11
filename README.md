# AI Code Assistant Commands

Custom commands for Claude Code and OpenAI Codex CLI.

## Supported Platforms

| Platform | Location | Invocation |
|----------|----------|------------|
| **Claude Code** | `.claude/commands/worktree.md` | `/worktree` |
| **Codex CLI** | `.codex/skills/worktree/SKILL.md` | `$worktree` |

## Commands

### Worktree Command

Create a git worktree with synced configs, content migration, and background dependency installation.

**Claude Code:**
```bash
/worktree [branch-name] [--stash] [--from <worktree>]
```

**Codex CLI:**
```bash
$worktree [branch-name] [--stash] [--from <worktree>]
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
7. **Config Sync** - Copies platform-specific configs
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

**Branch Name Handling:**

Branch names containing `/` are automatically converted to `-` for flat directory structure:

| Input | Output Directory |
|-------|------------------|
| `feature/auth` | `.worktrees/project/feature-auth/` |
| `bugfix/issue-123` | `.worktrees/project/bugfix-issue-123/` |
| `main` | `.worktrees/project/main/` |

**Content Migration Examples:**

```bash
# Scenario: You have uncommitted changes and need to switch context
/worktree hotfix-urgent --stash   # Claude Code
$worktree hotfix-urgent --stash   # Codex CLI

# Scenario: Pull changes from another worktree
/worktree feature-b --from feature-a   # Claude Code
$worktree feature-b --from feature-a   # Codex CLI
```

**Quick switch to new worktree:**

After the command completes, the launch command is copied to your clipboard.
Just press `Ctrl+C` to exit, then paste and run to start in the new worktree.

**Synced files:**

| File/Directory | Claude Code | Codex CLI |
|----------------|-------------|-----------|
| `.claude/` | ✓ | - |
| `.codex/` | - | ✓ |
| `AGENTS.md` | - | ✓ |
| `.env` / `.env.local` | ✓ | ✓ |
| `.vscode/` | ✓ | ✓ |
| `.cursorrules` / `.windsurfrules` | ✓ | ✓ |
| `docs/.local/` | ✓ | ✓ |

## Installation

### Claude Code

Copy `.claude/commands/` to your project root.

### Codex CLI

Copy `.codex/skills/` to your project root, or install to user scope:

```bash
cp -r .codex/skills/worktree ~/.codex/skills/
```

Restart Codex to activate the skill.

## References

- [Git Worktree Documentation](https://github.com/git/git/blob/master/Documentation/git-worktree.txt)
- [Claude Code Slash Commands](https://docs.anthropic.com/en/docs/claude-code)
- [Codex Skills Documentation](https://developers.openai.com/codex/skills)
- [Codex Create Skill Guide](https://developers.openai.com/codex/skills/create-skill/)

## License

MIT
