---
description: Summarise the current session into a durable handoff document for a future agent
argument-hint: [project_name]
---

# Handoff: Summarise work for a future agent

## Command Usage
`/handoff-pickup:handoff [project_name]`

## Purpose
Creates a timestamped handoff document that captures everything a future agent needs to pick up this work. Stored in `~/.claude/handoffs/` with a folder structure derived from the current repository.

## Instructions for Claude

### Step 1: Get the current session ID

Run this to get the current Claude Code session ID so the receiving agent can access the raw conversation history:

```bash
# Claude Code names each per-project transcript directory by encoding the
# working directory: every non-alphanumeric character becomes a dash.
# We match that encoding exactly, then pick the most recently modified session.
CONFIG_DIR="${CLAUDE_CONFIG_DIR:-$HOME/.claude}"
PROJECT_DIR=$(echo "$PWD" | sed 's|[^a-zA-Z0-9]|-|g')
SESSION_ID=$(ls -t "$CONFIG_DIR"/projects/"$PROJECT_DIR"/*.jsonl 2>/dev/null | head -1 | xargs basename 2>/dev/null | sed 's/\.jsonl$//')
echo "$SESSION_ID"
```

This is best-effort. It relies on Claude Code's internal transcript layout, which is not a documented public API, and it assumes a single active session per directory. If it returns empty (not in a project context, a different layout, or multiple concurrent sessions), set `SESSION_ID` to empty and skip the session line in the handoff rather than guessing.

### Step 2: Determine the project name

The argument `$ARGUMENTS` is the project name (snake_case).

- If provided, use it directly (convert to snake_case if needed)
- If not provided, ask the user (via AskUserQuestion if available, otherwise in plain text): "What should I call this handoff? Give me a short project name (e.g. auth_refactor, api_migration, test_fixes)"

### Step 3: Determine the repository context

Run this to build the folder path:

```bash
# Strip any trailing .git, then keep the last two path segments (org/repo).
NAMESPACE=$(git -C "$(pwd)" remote get-url origin 2>/dev/null \
  | sed -E 's#\.git$##' \
  | sed -E 's#.*[:/]([^/]+/[^/]+)$#\1#')
# Fall back to no-repo when not in a git repo, or when there is no origin
# remote (forks using upstream, local-only repos, custom-named remotes).
[ -z "$NAMESPACE" ] && NAMESPACE="no-repo"
echo "$NAMESPACE"
```

This gives you something like `your-org/your-repo`. Use it to build the storage path.

Note: deeply nested namespaces (e.g. GitLab subgroups) collapse to the last two segments. That is fine for storage, but if you rely on the exact path being unique, double-check for collisions.

### Step 4: Get the timestamp

```bash
date +"%Y%m%d_%H%M%S"
```

### Step 5: Build the file path

The handoff goes to:
```
~/.claude/handoffs/{namespace}/{project_name}/{timestamp}.md
```

Example: `~/.claude/handoffs/your-org/your-repo/auth_refactor/20260101_090000.md`

(If `CLAUDE_CONFIG_DIR` is set, use it in place of `~/.claude`.) Create any missing directories.

### Step 6: Write the handoff

Review the ENTIRE conversation. Then write a handoff document with this structure:

```markdown
# Handoff: {Project Name (human readable)}

> **Previous session**: `claude --resume {SESSION_ID}`
> Use this to access the full raw conversation history if you need more context than this handoff provides.

**Created**: {timestamp}
**Repository**: {namespace}
**Working directory**: {pwd}
**Branch**: {current git branch}

## What we're doing

One paragraph. What is the project, what is the goal.

## Where we got to

What's been completed. What state things are in right now. Be specific about files created, pages built, things deployed.

## Key decisions made

Bullet list. Each decision: what was decided, why, and what was rejected. These are the things a new agent MUST know to avoid re-litigating.

## Files that matter

List every file relevant to the project with a one-line description of what it is and whether it's current or outdated.

## Feedback received

Any feedback from team members, stakeholders, reviewers. Who said what, and what was done about it.

## What's left to do

Numbered list of remaining work, in priority order.

## Open questions

Anything unresolved that needs the user's input or team discussion.

## Context a new agent should read first

List of files the new agent should read before doing anything, in order. Be specific with full paths.
```

**Critical rules for writing the handoff:**
- Be comprehensive but concise. A new agent reading this should understand everything without reading the full conversation.
- Emphasise DECISIONS. The reasoning behind choices is the hardest thing to reconstruct.
- Include rejected alternatives for major decisions (e.g. "we chose X over Y because Z").
- Use full file paths, never relative.
- Don't be vague. "We updated the page" is useless. "We added a green callout with a toggle at the top of the settings page" is useful.

### Step 7: Confirm

Tell the user the handoff was written and give them the full path. Mention the key sections so they can verify nothing was missed.
