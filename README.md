# Worktree Command

Git worktree automation for Claude Code and Codex CLI with config sync, content migration, and background dependency installation.

## Features

- **Unified Directory Structure** - Organizes worktrees in `../.worktrees/<project>/`
- **Content Migration** - Transfer uncommitted changes between worktrees via `--stash` or `--from`
- **Config Sync** - Automatically copies `.env`, `.vscode/`, platform configs, etc.
- **Background Dependencies** - Detects package manager and installs in background
- **Clipboard Integration** - Copies launch command for quick worktree switching

## Quick Start

```bash
# Claude Code
/worktree feature-auth

# Codex CLI
$worktree feature-auth
```

Output:
```
Worktree created successfully!

  Path:     ../.worktrees/project/feature-auth/
  Branch:   feature-auth
  Configs:  .claude/, .env, .vscode/, ...

Quick start (copied to clipboard):
  cd ../.worktrees/project/feature-auth && claude
```

## Usage

### Basic

```bash
/worktree [branch-name]              # Create worktree for branch
/worktree                            # Interactive branch selection
```

### With Content Migration

```bash
/worktree feature --stash            # Migrate current uncommitted changes
/worktree feature --from hotfix      # Migrate changes from another worktree
```

### Options

| Option | Description |
|--------|-------------|
| `--stash` | Stash current changes and apply to new worktree |
| `--from <name>` | Stash changes from specified worktree and apply |

### Branch Name Handling

Branch names with `/` are converted to `-` for flat directory structure:

| Input | Output Directory |
|-------|------------------|
| `feature/auth` | `.worktrees/project/feature-auth/` |
| `bugfix/issue-123` | `.worktrees/project/bugfix-issue-123/` |

### Directory Structure

```
parent/
├── project/                    # Main repo
└── .worktrees/
    └── project/                # Grouped by project
        ├── feature-auth/
        ├── hotfix-123/
        └── ...
```

### Synced Files

| File/Directory | Claude Code | Codex CLI |
|----------------|:-----------:|:---------:|
| `.claude/` | ✓ | - |
| `.codex/` | - | ✓ |
| `AGENTS.md` | - | ✓ |
| `.env` / `.env.local` | ✓ | ✓ |
| `.vscode/` | ✓ | ✓ |
| `.cursorrules` / `.windsurfrules` | ✓ | ✓ |
| `docs/.local/` | ✓ | ✓ |

## Installation

### Claude Code

```bash
cp -r .claude/skills/worktree <your-project>/.claude/skills/
```

### Codex CLI

```bash
# Project scope
cp -r .codex/skills/worktree <your-project>/.codex/skills/

# Or user scope (available across all projects)
cp -r .codex/skills/worktree ~/.codex/skills/
```

Restart Codex to activate.

## Supported Platforms

| Platform | Location | Invocation |
|----------|----------|------------|
| Claude Code | `.claude/skills/worktree/` | `/worktree` or natural language |
| Codex CLI | `.codex/skills/worktree/` | `$worktree` |

## References

- [Git Worktree Documentation](https://github.com/git/git/blob/master/Documentation/git-worktree.txt)
- [Claude Code Slash Commands](https://docs.anthropic.com/en/docs/claude-code)
- [Codex Skills Documentation](https://developers.openai.com/codex/skills)

## License

MIT
