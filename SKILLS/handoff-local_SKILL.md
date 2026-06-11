---
name: handoff-local
description: Generate a comprehensive, resumable handoff summary of the current conversation and write it to a local markdown file at handoffs/{YYYY}/{MM}/{DD}/{time}_{slug}.md. Use this whenever the user issues the /handoff-local command (optionally followed by "focus on" a topic, or a bare slug), or when they ask to hand off, compact, summarize for a new context window, create a continuation summary, or otherwise capture the state of the current chat so work can resume later in a fresh conversation. Trigger even if they only say /handoff-local with no other words.
---

# Handoff (universal)

Generate a structured summary of the **current conversation** and write it to a local markdown file under `handoffs/`. The summary is designed so that you — or another agent, in a fresh context window — can resume the work where this conversation's history has been replaced by the summary.

This is a single, agent-agnostic command. The **core body** (Steps 1–3) is identical for every agent; only how you *register* the command and which *argument placeholder* you use differ per agent. See "Install for your agent" below, then follow the core body.

---

## Install for your agent

Save this file (or its core body) where your agent looks for custom commands. The name is always `handoff-local` to avoid colliding with any built-in `handoff` (invoked as `/handoff-local` in Claude/Gemini, `$handoff-local` or implicitly in Codex).

> **Codex tip:** install it as a **skill** (`~/.codex/skills/handoff-local/SKILL.md`), *not* a custom prompt. Codex's `~/.codex/prompts/` path is deprecated and will register this as a one-shot prompt instead of a skill — if it landed there, move the folder to `~/.codex/skills/` and restart Codex.

| Agent | Where to put it | Invoke | Argument token |
| --- | --- | --- | --- |
| **Claude Code** | `~/.claude/skills/handoff-local/SKILL.md` (or project `.claude/skills/handoff-local/SKILL.md`). Keep the YAML front matter above. | `/handoff-local` | `$ARGUMENTS` |
| **OpenAI Codex CLI** | Save as a **skill**: `~/.codex/skills/handoff-local/SKILL.md` (or project `.codex/skills/handoff-local/SKILL.md`). Keep the YAML front matter above — same `SKILL.md` format as Claude. **Not** the deprecated `~/.codex/prompts/` location. | `$handoff-local` (or implicit) | `$ARGUMENTS` |
| **Gemini CLI** | `~/.gemini/commands/handoff-local.toml` (or project `.gemini/commands/`). Wrap the core body in the TOML snippet below, then run `/commands reload`. | `/handoff-local` | `{{args}}` |
| **Cursor / generic agent** | Add the core body as a custom rule/prompt, or just paste it into chat. Pass the topic text directly. | per your tool | the text you typed after the command |

> **Placeholder note:** wherever the core body says `$ARGUMENTS`, **Gemini CLI** users should read it as `{{args}}`, and a **generic agent** should read it as "the text the user typed after the command." The instructions are otherwise unchanged.

### Gemini CLI wrapper (ready to paste into `handoff-local.toml`)

```toml
description = "Write a resumable handoff summary of this conversation to handoffs/{YYYY}/{MM}/{DD}/{time}_{slug}.md"
prompt = """
Treat the text after the command as the arguments (referred to below as $ARGUMENTS): {{args}}

<paste the entire "Core body" section below here>
"""
```

---

## Core body

The command is invoked as:

```
/handoff-local
/handoff-local <slug>
/handoff-local focus on <topic>
```

Treat the text after `/handoff-local` as `$ARGUMENTS` (Gemini: `{{args}}`).

### Step 1 — Choose the prompt and generate the summary

If `$ARGUMENTS` begins with `focus on ` (e.g. `/handoff-local focus on hermes-memory`), use the **Detailed prompt**. Otherwise (no args, or a bare slug) use the **Continuation prompt**.

Apply the chosen prompt to the current conversation and produce the summary.

#### Continuation prompt (default)

> You have been working on the task discussed in this conversation but have not yet completed it. Write a continuation summary that will allow you (or another agent) to resume work efficiently in a future context window where the conversation history will be replaced with this summary. Your summary should be structured, concise, and actionable. Include:
>
> 1. **Task Overview** — The user's core request and success criteria; any clarifications or constraints they specified.
> 2. **Current State** — What has been completed so far; files/artifacts created, modified, or analyzed (with paths if relevant); key outputs produced.
> 3. **Important Discoveries** — Technical constraints or requirements uncovered; decisions made and their rationale; errors encountered and how they were resolved; approaches that were tried and didn't work (and why).
> 4. **Next Steps** — Specific actions needed to complete the task; blockers or open questions; priority order if multiple steps remain.
> 5. **Context to Preserve** — User preferences or style requirements; non-obvious domain-specific details; any promises made to the user.
>
> Be concise but complete — err on the side of including information that would prevent duplicate work or repeated mistakes. Write so the task can be resumed immediately. Wrap your summary in `<summary>` tags.

#### Detailed prompt (when `$ARGUMENTS` begins with `focus on `)

> Create a detailed summary of this conversation. This summary will be placed at the start of a continuing session; newer messages that build on this context will follow after your summary (you do not see them here). Summarize thoroughly so that someone reading only your summary and then the newer messages can fully understand what happened and continue the work.
>
> Before the final summary, wrap your analysis in `<analysis>` tags. In your analysis:
>
> 1. Chronologically analyze each message and section of the conversation. For each, identify: the user's explicit requests and intents; your approach to addressing them; key decisions, technical concepts, and patterns; specific details (file names, full code snippets, function signatures, file edits); errors encountered and how they were fixed; specific user feedback, especially where the user told you to do something differently. Note any security-relevant instructions or constraints the user stated (sensitive files/data to avoid, operations that must not be performed, credential or secret handling rules) — these MUST be preserved **verbatim** so they continue to apply after compaction.
> 2. Double-check for technical accuracy and completeness.
>
> Then provide the summary with these sections:
>
> 1. **Primary Request and Intent** — The user's explicit requests and intents, in detail.
> 2. **Key Technical Concepts** — Important concepts, technologies, and frameworks discussed.
> 3. **Files and Code Sections** — Specific files/sections examined, modified, or created; full code snippets where applicable; why each matters.
> 4. **Errors and Fixes** — Errors encountered and how they were fixed.
> 5. **Problem Solving** — Problems solved and ongoing troubleshooting.
> 6. **All User Messages** — List ALL non-tool-result user messages. Preserve security-relevant instructions or constraints **verbatim**.
> 7. **Pending Tasks** — Outstanding tasks.
> 8. **Work Completed** — What was accomplished by the end of this portion.
> 9. **Context for Continuing Work** — Context, decisions, or state needed to continue.
>
> Ensure precision and thoroughness.

---

### Step 2 — Determine the topic slug

- If `$ARGUMENTS` begins with `focus on `, strip that prefix and kebab-case the remainder (e.g. `focus on hermes memory` → `hermes-memory`).
- Otherwise, if `$ARGUMENTS` is non-empty, use it verbatim as the slug (already kebab-case).
- Otherwise, derive a 2–4 word kebab-case slug from the main topic (e.g. `hermes-fts-fix`, `docker-compose-refactor`).

---

### Step 3 — Write the handoff to a local markdown file

Save the summary to a file in the current project rather than printing it to chat:

> **Important — where "project root" is:** the project root is the working directory of the *current conversation* (confirm with `pwd`), **not** the directory where this command/skill file itself lives. Never write the handoff inside your agent's config folder (`.claude/`, `.codex/`, `.gemini/`, `.cursor/`, etc.).

1. Determine today's date from the current date in context, then split it into three separate, zero-padded components: four-digit year `YYYY`, two-digit month `MM`, and two-digit day `DD` (e.g. `2026`, `06`, `09`).
2. Determine the current **local** time in 24-hour format as `HH-MM` — a two-digit hour and two-digit minute separated by a hyphen (e.g. `14-11` for 2:11 PM, `09-05` for 9:05 AM). If the exact local time is not available from context, obtain it (for example via a shell command such as `date +%H-%M`).
3. Build the path `handoffs/{YYYY}/{MM}/{DD}/{time}_{slug}.md`, relative to the project root. This is a strict **three-tier nested tree**: year, month, and day are each their own subdirectory — never join them with hyphens into a single `YYYY-MM-DD` folder. Use forward slashes as path separators; your file-writing tool resolves them correctly on both Windows and POSIX, so do not hand-build backslash paths. The `{time}` prefix and `{slug}` are joined by an underscore. Example: `handoffs/2026/06/09/14-11_hermes-fts-fix.md`.
4. Write the complete summary from Step 1 — including its `<summary>` or `<analysis>`/summary sections — to that file using your file-writing tool. Create the full `handoffs/{YYYY}/{MM}/{DD}/` directory chain if it does not already exist.
5. If a file with that name already exists, append a short numeric suffix to the slug (e.g. `{time}_{slug}-2.md`) so an earlier handoff is never overwritten.

The file body is the raw summary markdown itself — no surrounding code fence and no extra preamble.

After writing the file, report only a brief confirmation with the relative path, for example:

> Handoff written to `handoffs/2026/06/09/14-11_hermes-fts-fix.md`.

Add no other commentary — the saved file is the deliverable.
