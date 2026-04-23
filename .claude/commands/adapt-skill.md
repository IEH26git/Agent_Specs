---
name: adapt-skill
description: Adapts a skill file from another Claude project to work for the Agent_Specs agent, then deletes the source file.
---

Adapt the skill file provided by the user to work for this Agent_Specs agent. Follow these steps exactly.

## Step 1 — Identify the source file

The user will either:
- Pass a file path as an argument to this command, or
- Have dropped a `.md` file somewhere in this project directory.

If no path was given as an argument, run `find /Users/ianheiman/_Agents/_All-Agents-admin/Agent_Specs -name "*.md" -newer /Users/ianheiman/_Agents/_All-Agents-admin/Agent_Specs/.claude/commands/adapt-skill.md -not -path "*/specs/*" -not -path "*/raw/*" -not -path "*/.claude/*"` to locate recently added `.md` files and ask the user to confirm which one is the source skill.

Read the source file completely before doing anything else.

## Step 2 — Analyze the skill

Identify:
- What the skill does (its purpose and steps)
- Any project-specific references that need changing: agent name, directory paths, memory layer references, tool names, scope assumptions, output locations, or other project identifiers
- Whether the skill's purpose is in scope for Agent_Specs (curating, storing, summarizing, and analyzing agent specification files). If it is clearly out of scope, tell the user and stop.

## Step 3 — Adapt the skill

Produce a modified version of the skill with these changes applied:

**Identity and paths:**
- Replace any other agent name or nickname with `Agent_Specs`
- Update all directory paths to reflect this project root: `/Users/ianheiman/_Agents/_All-Agents-admin/Agent_Specs`
- Work-in-progress specs go to `draft/`; final public-ready specs go to `specs/`; source material is read-only in `raw/`

**Memory layer references:**
- Active session context → `.claude/memory.md`
- Structured local facts → `claude-local-memory.db` (SQLite)
- Cross-tool thought captures → Open Brain MCP (`mcp__open-brain__capture_thought`, etc.)
- Persistent preferences/profile → auto-memory at `~/.claude/projects/.../memory/`

**Prose and formatting rules (apply to any output the skill generates):**
- No sentence fragments
- No em-dashes connecting two independent clauses
- No parallelisms ("it's not X, it's Y") — use direct assertions
- Tables: left-justify all cells except number/currency columns (right-justify those); header alignment matches column data alignment

**Frontmatter:**
- Ensure the `name:` field matches the destination filename (without `.md`)
- Keep or write a clear one-line `description:`

**Scope guard:**
- If the original skill had a scope check for a different domain, update it to check for agent specification relevance instead.

Do not add features or steps that weren't in the original. Do not remove steps unless they are entirely inapplicable and have no Agent_Specs equivalent.

## Step 4 — Write the adapted skill

Save the adapted skill to `.claude/commands/<name>.md` where `<name>` matches the `name:` frontmatter field.

If a file already exists at that path, show the user the diff and ask whether to overwrite before writing.

## Step 5 — Confirm and delete the source file

Show the user:
1. The path where the adapted skill was saved
2. A brief plain-English summary of what was changed

Then delete the source file. Do not ask for confirmation before deleting — the user requested this behavior when they invoked this skill.

After deleting, confirm: "Source file deleted: `<path>`."
