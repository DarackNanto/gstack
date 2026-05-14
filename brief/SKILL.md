---
name: brief
version: 1.0.0
description: |
  End-of-session knowledge capture. Reads CLAUDE.md and project memory files,
  extracts new facts, decisions, and conventions from the current conversation,
  then proposes specific updates one at a time. Keeps CLAUDE.md and memory
  current without the user having to narrate what changed. Run at the end of
  any session where real decisions were made.
triggers:
  - /brief
  - update memory
  - capture decisions
  - end of session
  - save what we decided
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - AskUserQuestion
---

# /brief

Capture session decisions and propose targeted updates to CLAUDE.md and project memory.

## Step 1 — Read the current state

Read in parallel:

1. `CLAUDE.md` in the current project root (if it exists)
2. The project memory index: `~/.claude/projects/<slug>/memory/MEMORY.md`
3. Any memory files referenced in MEMORY.md that are likely relevant to this session's topic

Find the project slug:
```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" 2>/dev/null || true
echo "SLUG: ${SLUG:-unknown}"
MEMORY_DIR="${GSTACK_HOME:-$HOME/.gstack}/../../.claude/projects/${SLUG:-unknown}/memory"
# Standard claude memory path:
MEMORY_DIR="$HOME/.claude/projects/${SLUG:-unknown}/memory"
echo "MEMORY_DIR: $MEMORY_DIR"
ls "$MEMORY_DIR" 2>/dev/null || echo "(no memory files yet)"
```

If no CLAUDE.md and no memory directory exist, note that and continue — new projects need their first entries too.

## Step 2 — Extract new information from this session

Review the conversation context carefully. Identify everything that was established, decided, corrected, or confirmed in this session that is NOT already captured in CLAUDE.md or the memory files.

Categorise each item as one of:

| Category | Goes in |
|---|---|
| Company-wide convention, rule, or architecture decision | CLAUDE.md |
| Division or product structure change | CLAUDE.md (relevant division section) |
| Fact about the user's role, preferences, working style | memory — type: user |
| Guidance on how to approach work, what to avoid or repeat | memory — type: feedback |
| Ongoing project state, goals, initiatives, who/what/when | memory — type: project |
| Pointer to an external resource | memory — type: reference |

**Do not extract:**
- Code patterns or file paths derivable from the codebase
- Git history or who changed what
- Debugging solutions or fix recipes
- Anything already in CLAUDE.md or memory
- Ephemeral task state (in-progress work, this session's TODOs)

If nothing new was established in this session, say so and stop. Do not invent updates.

## Step 3 — Draft proposed changes

For each item identified in Step 2, draft the exact text that would be added or changed:

- For CLAUDE.md: the exact markdown paragraph, table row, or section to add/modify, and exactly where it goes
- For a new memory file: the full file content including frontmatter
- For an existing memory file: the specific lines to add or change

Group related items. If three items all belong in the same memory file, propose them together as one update.

## Step 4 — Present each proposed change via AskUserQuestion

For each proposed change (or group), ask:

```
D<N> — <one-line title of what this update captures>
Project/branch/task: <grounding sentence>
ELI10: <plain explanation of what this change captures and why it matters>
Stakes if we pick wrong: <what a future agent gets wrong if this is missing or wrong>
Recommendation: A because this is what was decided
Note: options differ in kind, not coverage — no completeness score.
A) Add as proposed (recommended)
  ✅ Captures [specific decision/fact] so future agents start with correct context
  ✅ Prevents re-explaining [specific thing] in every new session
  ❌ Adds length to CLAUDE.md / memory — only worth it if this will recur
B) Edit before adding
  ✅ User can correct wording or scope before it is written
  ✅ Avoids locking in a subtly wrong framing
  ❌ Requires the user to provide the corrected text
C) Skip
  ✅ Keeps CLAUDE.md / memory lean
  ❌ This decision will need to be re-explained in future sessions
Net: capturing real decisions saves future re-work; capturing ephemeral state is noise
```

If the user chooses B, ask them for the corrected text, then write that version.

## Step 5 — Write approved changes

For each approved item:

- **CLAUDE.md addition**: Use Edit to insert at the correct location, or append a new section
- **New memory file**: Use Write to create the file, then Edit MEMORY.md to add the index entry
- **Existing memory file update**: Use Edit to modify the relevant section

After writing each change, confirm the file was saved correctly by reading the affected lines.

## Step 6 — Summary

```
/brief — COMPLETE
═══════════════════════════════════════════════
Updated CLAUDE.md:
  ✓ [description of each change]

Updated memory:
  ✓ [filename] — [description]

Skipped:
  — [item and reason if user skipped]

Nothing changed:
  — [item if already captured]
═══════════════════════════════════════════════
```

If nothing was updated, say: "Nothing new to capture — CLAUDE.md and memory are already current."
