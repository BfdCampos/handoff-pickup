---
description: Find and load a previous handoff document so you can resume interrupted work
argument-hint: [search_terms]
---

# Pickup: Resume work from a handoff

## Command Usage
`/handoff-pickup:pickup [search_terms]`

## Purpose
Finds and loads a previous handoff document (written by `/handoff-pickup:handoff`) so you can continue where a previous agent left off.

## Instructions for Claude

### Step 0: Check any handoffs exist

```bash
CONFIG_DIR="${CLAUDE_CONFIG_DIR:-$HOME/.claude}"
if [ ! -d "$CONFIG_DIR/handoffs" ] || [ -z "$(find "$CONFIG_DIR/handoffs" -name '*.md' -type f 2>/dev/null)" ]; then
  echo "NO_HANDOFFS"
fi
```

If this prints `NO_HANDOFFS`, tell the user there are no handoffs yet and that they can create one with `/handoff-pickup:handoff`, then stop. Do not continue.

### Step 1: Find the handoff

The argument `$ARGUMENTS` contains search terms (e.g. "auth refactor", "api migration").

**If search terms are provided:**

1. First, try to match against folder names in the handoffs directory:
   ```bash
   find "$CONFIG_DIR/handoffs" -maxdepth 4 -type d 2>/dev/null
   ```

2. Look for folder names that fuzzy-match the search terms. If there's a clear match, list the handoff files in that folder (there may be multiple timestamps, take the most recent).

3. If no folder match, search inside handoff files for the terms:
   ```bash
   grep -rl "search terms" "$CONFIG_DIR/handoffs" 2>/dev/null
   ```

**If no search terms are provided, or no match found:**

1. List all available handoffs:
   ```bash
   find "$CONFIG_DIR/handoffs" -name "*.md" -type f 2>/dev/null | sort
   ```

2. Present the options to the user and ask which to pick up. Group them by namespace (the `{org}/{repo}` or `no-repo` folder) and show the latest timestamp per project. If AskUserQuestion is available, offer each project as a short option (one option per project, labelled with the project name and namespace); otherwise print a numbered list and ask the user to reply with a number or name. For example, the projects to offer might be:

   - `auth_refactor` (acme/webapp, latest 20260101_090000)
   - `api_migration` (acme/webapp, latest 20251231_143000)
   - `blog_redesign` (acme/marketing-site, latest 20251230_091500)

   Keep labels short; the namespace is context, the project name is the choice.

### Step 2: Read the handoff

Once you've identified the correct handoff file, read it completely.

### Step 3: Read the context files

The handoff has a "Context a new agent should read first" section. Read every file listed there, in order. These are essential for understanding the project.

### Step 4: Confirm and orient

Tell the user:
- What project you've picked up
- When the handoff was created
- A brief summary of where things left off (2-3 sentences)
- What the next steps are
- Ask if anything has changed since the handoff was written

If the handoff contains a session ID (in the "Previous session" line), also tell the user:
> If you'd rather continue the exact conversation instead of starting fresh with a new agent, run: `claude --resume <session_id>`
>
> (This is best-effort. The session may not resume if it was written on a different machine or an older Claude Code version.)

**Do NOT start doing work yet.** Wait for the user to confirm or update you on what's changed.
