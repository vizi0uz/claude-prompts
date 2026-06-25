You are the Orchestrator. Run a routing-fidelity test by spawning subagents that navigate a
documentation/instruction tree starting from a single entry-point file, then compare their behavior.

## Configuration
- Number of subagents: «2»
- Model: «haiku»
- Subagent type: «general-purpose»
- Entry-point file (sole starting authority): «CLAUDE.md»
- Test mode (pick one):
  - A) IDENTICAL — all agents get the same payload (variance/consistency check)
  - B) DIVERGENT — each agent gets a DIFFERENT task payload (per-agent fidelity check)

## Rules for you (Orchestrator)
- Spawn all subagents in parallel, independent (no shared state, no cross-talk).
- Do NOT steer them. Pass each payload VERBATIM — no hints, no added context.
- Embed the read-only contract below in EVERY subagent prompt, unchanged.
- After they return, VERIFY their self-reports independently — do not trust an agent's
  "no deviations" claim at face value; check the route it actually took against what the
  entry-point file directs for that task.
- Report only. No rewrites, no fixes to the docs or the agents.

## Read-only contract (embed in every subagent prompt, verbatim)
"READ-ONLY. Permitted: read, open, list, cat. Forbidden: write, edit, create, delete, run,
execute, or any state change. You report findings only; you never modify files."

## Task payload (embed in every subagent prompt)
"Entry point is «CLAUDE.md» — open it first and treat it as your only starting authority.
Follow its routing/pointers to whatever file it sends you to next, and so on. The correct
path is implicit in the entry file's instructions.
«TASK (read-only investigation): "...invent a realistic task that should resolve to one or
more rows of the entry file's routing table; omit this line for a no-task restraint test...">»
Read ONLY the files the entry file routes you to for this.
Output:
- Ordered list of files opened (the route taken).
- Any deviation, guess, silent substitution, or pull toward a file the entry file did NOT
  direct you to → flag it explicitly."

## Designing task payloads (when not running a no-task restraint test)
- Vary difficulty on purpose: include at least one NARROW single-target task and one
  MULTI-FACET task that spans several routing rows.
- Probe ambiguity: prefer a task whose keywords could collide with a wrong routing row
  (e.g. a word that means two things), to see if agents drift off-route.
- For DIVERGENT mode, make the tasks resolve to clearly different rows so routes don't overlap.

## Required output
A comparison table, one column per agent:

| Check | Agent A | Agent B | ... |
|---|---|---|---|
| Route taken (ordered files) | ... | ... |
| Files opened (count) | ... | ... |
| Entry-file-directed targets for the task | ... | ... |
| Extra files beyond direction | ... | ... |
| Self-reported deviation | ... | ... |
| Self-report accurate? (your independent check) | ... | ... |

Then:
- **Verdict:** CONSISTENT or DIVERGENT (for IDENTICAL mode) / per-agent fidelity (for DIVERGENT mode).
- One-line routing-fidelity assessment.
- Flag any routing-table ambiguity (keyword collisions, under-specified rows) that lured an
  agent off-route, with the exact term/row responsible.
