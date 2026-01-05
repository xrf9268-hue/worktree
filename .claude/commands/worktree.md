---
description: Create a git worktree with synced configs and background dependency installation
argument-hint: [branch-name]
allowed-tools: Bash(git:*), Bash(cp:*), Bash(mkdir:*), Bash(ls:*), Bash(cat:*), Bash(cd:*), Bash(pnpm:*), Bash(npm:*), Bash(yarn:*), Bash(bun:*), Bash(echo:*), Bash(command:*), Bash(pbcopy:*), Bash(xclip:*), Bash(xsel:*), Bash(wl-copy:*)
---

# Git Worktree Workflow

You are an expert DevOps assistant. Create a parallel development environment ready for coding immediately.

## Step 1: Context & Path Analysis

Execute these commands to understand the current environment:
```bash
git rev-parse --show-toplevel
git rev-parse --git-common-dir
```

- Identify if we're in the **main repo** or an **existing worktree**
- The "Main Repo" is the source of config files (use `--git-common-dir` to find it)

## Step 2: Branch Resolution

**If argument provided (`$ARGUMENTS`)**: Use it as the target branch name.

**If NO argument provided**:
1. List recent branches:
   ```bash
   git for-each-ref --sort=-committerdate refs/heads/ --format='%(refname:short)' | head -n 10
   ```
2. Ask the user to select a branch or enter a new one.

## Step 3: Worktree Creation

- Determine the new folder path (sibling to current repo root, e.g., `../project-branch-name`)
- Check if the branch exists locally or remotely:
  ```bash
  git branch --list <branch-name>
  git ls-remote --heads origin <branch-name>
  ```
- Create the worktree:
  - **Branch exists**: `git worktree add <new-path> <branch-name>`
  - **New branch**: `git worktree add -b <branch-name> <new-path>`

## Step 4: Environment Synchronization

Copy configuration files from the **Main Repo** to the **New Worktree** using `cp -r`.

**Target files/folders (copy only if they exist)**:
- `.claude/` (Claude Code settings and commands)
- `.env` and `.env.local` (environment variables)
- `docs/.local/` (local documentation)
- `.cursorrules` or `.windsurfrules` (editor rules)
- `.vscode/` (VS Code settings)

Use this pattern to safely copy:
```bash
[ -e "<main-repo>/.claude" ] && cp -r "<main-repo>/.claude" "<new-path>/"
```

## Step 5: Dependency Installation (Background)

Detect the package manager and install dependencies in the background:

1. **Detect**: Check for lockfiles in order:
   - `pnpm-lock.yaml` → pnpm
   - `yarn.lock` → yarn
   - `bun.lockb` → bun
   - `package-lock.json` → npm

2. **Install in background** (fire and forget):
   ```bash
   (cd <new-path> && <install-cmd> > /dev/null 2>&1 &)
   ```

3. Notify the user that installation is running in the background.

## Step 6: Copy Launch Command to Clipboard

Copy the command to launch Claude Code in the new worktree to the system clipboard.

**Cross-platform clipboard detection** (try in order):
```bash
# Detect available clipboard tool
if command -v pbcopy &> /dev/null; then
  CLIP_CMD="pbcopy"
elif command -v xclip &> /dev/null; then
  CLIP_CMD="xclip -selection clipboard"
elif command -v xsel &> /dev/null; then
  CLIP_CMD="xsel --clipboard --input"
elif command -v wl-copy &> /dev/null; then
  CLIP_CMD="wl-copy"
else
  CLIP_CMD=""
fi
```

**Copy the launch command**:
```bash
if [ -n "$CLIP_CMD" ]; then
  echo "cd <new-path> && claude" | $CLIP_CMD
  CLIPBOARD_SUCCESS=true
else
  CLIPBOARD_SUCCESS=false
fi
```

## Step 7: Summary Output

**Do NOT automatically open an editor.** Instead, provide a clear summary:

```
Worktree created successfully!

  Path: <new-path>
  Branch: <branch-name>
  Configs synced: .claude/, .env, .vscode/, ...
  Dependencies: Installing in background (pnpm/npm/yarn/bun)
```

**If clipboard copy succeeded**, show:
```
Quick start (command copied to clipboard):
  1. Press Ctrl+C to exit current session
  2. Press Cmd+V (macOS) or Ctrl+Shift+V (Linux) to paste and run

Or manually:
  cd <new-path> && claude
```

**If clipboard copy failed**, show:
```
To start Claude Code in the new worktree:
  cd <new-path> && claude
```

This allows the user to quickly switch to the new worktree.
