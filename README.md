# handoff-pickup

Two Claude Code slash commands for carrying work across sessions.

Long sessions end. Context windows fill up, laptops sleep, you swap machines, or you simply want to start fresh without dragging a huge transcript along. When that happens the expensive thing to lose is not the code (that is on disk) but the reasoning: the decisions made, the alternatives rejected, the feedback received, and what was left to do.

This plugin gives you a durable place to put that:

- `/handoff-pickup:handoff` reviews the whole session and writes a structured markdown document capturing the state of the work.
- `/handoff-pickup:pickup` finds a previous handoff and loads it (plus the files it points at) so a brand new session can carry on.

Think of it as a save point for a piece of work, readable by both you and the next agent.

## How it works

Handoffs are stored as markdown under `~/.claude/handoffs/`, in a folder tree derived from the current git remote:

```
~/.claude/handoffs/{org}/{repo}/{project_name}/{timestamp}.md
```

For example `~/.claude/handoffs/your-org/your-repo/auth_refactor/20260101_090000.md`. Work that is not in a git repository is filed under `no-repo/` instead.

Because each handoff is a plain file, `/pickup` can find them later by folder name or by searching their contents, and you can read or edit them yourself at any time.

## Install

```
/plugin marketplace add BfdCampos/handoff-pickup
/plugin install handoff-pickup@bfdcampos
```

Then use the commands as `/handoff-pickup:handoff` and `/handoff-pickup:pickup`.

## Usage

Create a handoff at the end of a working session:

```
/handoff-pickup:handoff auth_refactor
```

If you leave off the project name it will ask for one. The command reviews the conversation and writes a document covering what you are doing, where things got to, the key decisions (and what was rejected), the files that matter, feedback received, what is left, open questions, and which files a new agent should read first.

Later, in a fresh session, pick it back up:

```
/handoff-pickup:pickup auth refactor
```

It fuzzy-matches your search terms against existing handoffs, loads the most recent match, reads the context files it lists, and gives you a short orientation before waiting for you to confirm. Run it with no arguments to see everything available.

## The resume line

Each handoff includes a best-effort `claude --resume <session_id>` line so you can reopen the exact original conversation if the summary is not enough. This depends on Claude Code's internal transcript layout, which is not a documented API, so treat it as a convenience rather than a guarantee. It may be absent or point elsewhere if you ran multiple sessions in the same directory, are on a different machine, or are on an older Claude Code version. The handoff document itself is the source of truth.

## Limitations and notes

- Storage respects `CLAUDE_CONFIG_DIR` if you have set it, and otherwise defaults to `~/.claude`.
- The namespace is taken from the last two segments of your `origin` remote, so deeply nested namespaces (for example GitLab subgroups) collapse to `{subgroup}/{repo}`. Fine for storage, worth a glance if you need the path to be globally unique.
- Repositories without an `origin` remote (some forks, local-only repos) are filed under `no-repo/`.
- These are instructions executed by Claude, not a rigid script, so the exact wording of any given handoff will vary. The structure is what stays consistent.

## Requirements

Claude Code. No other dependencies.

## Licence

MIT. See [LICENSE](LICENSE).
