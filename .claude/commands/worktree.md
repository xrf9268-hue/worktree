---
description: Create a git worktree with synced configs, content migration, and background dependency installation
argument-hint: [branch-name] [--stash] [--from <worktree>]
allowed-tools: Bash(git:*), Bash(cp:*), Bash(mkdir:*), Bash(ls:*), Bash(cat:*), Bash(cd:*), Bash(pnpm:*), Bash(npm:*), Bash(yarn:*), Bash(bun:*), Bash(echo:*), Bash(command:*), Bash(pbcopy:*), Bash(xclip:*), Bash(xsel:*), Bash(wl-copy:*), Bash(date:*), Bash(grep:*)
---

# Git Worktree Workflow

You are an expert DevOps assistant. Create a parallel development environment with optional content migration.

## Step 1: Argument Parsing

Parse `$ARGUMENTS` to extract:
- `BRANCH_NAME`: The target branch (first positional argument)
- `--stash`: Flag to migrate current uncommitted changes to the new worktree
- `--from <name>`: Migrate uncommitted changes from the specified worktree

Example parsing:
```bash
BRANCH_NAME=""
USE_STASH=false
FROM_WORKTREE=""

for arg in $ARGUMENTS; do
  if [ "$arg" = "--stash" ]; then
    USE_STASH=true
  elif [ "$prev_arg" = "--from" ]; then
    FROM_WORKTREE="$arg"
  elif [ "$arg" != "--from" ]; then
    [ -z "$BRANCH_NAME" ] && BRANCH_NAME="$arg"
  fi
  prev_arg="$arg"
done
```

## Step 2: Context & Path Analysis

Execute these commands to understand the current environment:
```bash
git rev-parse --show-toplevel
git rev-parse --git-common-dir
```

- Identify the **Main Repo** (use `--git-common-dir` to find it)
- Get the **Project Name** from the repo root basename

## Step 3: Branch Resolution

**If BRANCH_NAME provided**: Use it as the target branch.

**If NO BRANCH_NAME provided**:
1. List recent branches:
   ```bash
   git for-each-ref --sort=-committerdate refs/heads/ --format='%(refname:short)' | head -n 10
   ```
2. Ask the user to select a branch or enter a new one.

## Step 4: Worktree Path Determination

Use the unified `.worktrees/` directory structure:

```bash
PROJECT_ROOT=$(git rev-parse --show-toplevel)
PROJECT_NAME=$(basename "$PROJECT_ROOT")
PARENT_DIR=$(dirname "$PROJECT_ROOT")

# Unified worktrees directory
WORKTREES_BASE="${PARENT_DIR}/.worktrees/${PROJECT_NAME}"

# Create base directory if needed
mkdir -p "$WORKTREES_BASE"

# New worktree path
NEW_PATH="${WORKTREES_BASE}/${BRANCH_NAME}"
```

**Directory Structure:**
```
parent/
├── project/                    # Main repo
└── .worktrees/
    └── project/                # Grouped by project
        ├── feature-auth/       # Worktree 1
        ├── hotfix-123/         # Worktree 2
        └── ...
```

## Step 5: Content Migration (if requested)

### Option A: `--stash` flag

Migrate current uncommitted changes to the new worktree:

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

### Option B: `--from <worktree>` flag

Migrate changes from another worktree:

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

## Step 6: Worktree Creation

Check if branch exists and create the worktree:

```bash
# Check if branch exists locally or remotely
LOCAL_EXISTS=$(git branch --list "$BRANCH_NAME")
REMOTE_EXISTS=$(git ls-remote --heads origin "$BRANCH_NAME" 2>/dev/null)

if [ -n "$LOCAL_EXISTS" ] || [ -n "$REMOTE_EXISTS" ]; then
  # Branch exists
  git worktree add "$NEW_PATH" "$BRANCH_NAME"
else
  # New branch
  git worktree add -b "$BRANCH_NAME" "$NEW_PATH"
fi
```

## Step 7: Apply Migrated Changes

After worktree creation, apply the stashed changes:

```bash
# Apply stash if created
if [ "$STASH_CREATED" = true ] || [ "$FROM_STASH_CREATED" = true ]; then
  git -C "$NEW_PATH" stash apply "stash@{0}"
  echo "Changes applied to new worktree"

  # Note: Original stash is preserved for safety
  echo "Original stash preserved (use 'git stash drop' to remove)"
fi
```

## Step 8: Environment Synchronization

Copy configuration files from the **Main Repo** to the **New Worktree**:

```bash
MAIN_REPO=$(git rev-parse --git-common-dir | sed 's|/\.git.*||')

# Copy configs (only if they exist)
[ -e "$MAIN_REPO/.claude" ] && cp -r "$MAIN_REPO/.claude" "$NEW_PATH/"
[ -e "$MAIN_REPO/.env" ] && cp "$MAIN_REPO/.env" "$NEW_PATH/"
[ -e "$MAIN_REPO/.env.local" ] && cp "$MAIN_REPO/.env.local" "$NEW_PATH/"
[ -e "$MAIN_REPO/docs/.local" ] && mkdir -p "$NEW_PATH/docs" && cp -r "$MAIN_REPO/docs/.local" "$NEW_PATH/docs/"
[ -e "$MAIN_REPO/.cursorrules" ] && cp "$MAIN_REPO/.cursorrules" "$NEW_PATH/"
[ -e "$MAIN_REPO/.windsurfrules" ] && cp "$MAIN_REPO/.windsurfrules" "$NEW_PATH/"
[ -e "$MAIN_REPO/.vscode" ] && cp -r "$MAIN_REPO/.vscode" "$NEW_PATH/"
```

## Step 9: Dependency Installation (Background)

Detect package manager and install dependencies:

```bash
cd "$NEW_PATH"

if [ -f "pnpm-lock.yaml" ]; then
  (pnpm install > /dev/null 2>&1 &)
  PKG_MGR="pnpm"
elif [ -f "yarn.lock" ]; then
  (yarn install > /dev/null 2>&1 &)
  PKG_MGR="yarn"
elif [ -f "bun.lockb" ]; then
  (bun install > /dev/null 2>&1 &)
  PKG_MGR="bun"
elif [ -f "package-lock.json" ]; then
  (npm install > /dev/null 2>&1 &)
  PKG_MGR="npm"
else
  PKG_MGR=""
fi
```

## Step 10: Copy Launch Command to Clipboard

```bash
# Detect clipboard tool
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

# Copy launch command
if [ -n "$CLIP_CMD" ]; then
  echo "cd $NEW_PATH && claude" | $CLIP_CMD
  CLIPBOARD_SUCCESS=true
else
  CLIPBOARD_SUCCESS=false
fi
```

## Step 11: Summary Output

**Do NOT automatically open an editor.** Provide a clear summary:

```
Worktree created successfully!

  Path:     ../.worktrees/<project>/<branch>/
  Branch:   <branch-name>
  Configs:  .claude/, .env, .vscode/, ...
```

**If content was migrated**, add:
```
  Migrated: ✓ Changes applied from [current|<source-worktree>]
            Original stash preserved for safety
```

**If dependencies detected**, add:
```
  Dependencies: Installing in background (<pkg-mgr>)
```

**Quick start section:**

If clipboard succeeded:
```
Quick start (copied to clipboard):
  1. Press Ctrl+C to exit current session
  2. Paste and run: cd <path> && claude
```

If clipboard failed:
```
To start Claude Code in the new worktree:
  cd <path> && claude
```
