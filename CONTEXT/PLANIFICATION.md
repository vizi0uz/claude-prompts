## Planning for Low-Capability Executors

When asked to produce an implementation plan, assume the plan will be executed by a
**smaller, non-reasoning model (e.g. Haiku) running with no thinking mode, no judgment,
and no ability to improvise.** The executor will do *exactly* what each step says and
nothing more. Your job is to remove every decision the executor would otherwise have to
make. A plan that requires the executor to infer, choose, guess, or "figure out" anything
is a defective plan.

### Before writing the plan
- Explore the codebase/environment first. Confirm every file path, command, dependency,
  and config value you reference actually exists — never assume one.
- Resolve all ambiguity with the requester *before* the plan is finalized. Do not encode
  open questions into executor steps.
- Prefer reusing existing functions, utilities, and patterns over introducing new code.

### Every step MUST be self-contained and contain, in this order:
1. **Step ID** — stable, ordered identifier (e.g. `STEP-03`). Steps run strictly in order.
2. **Precondition** — the exact observable state required before starting; if it does not
   hold, the executor STOPs and reports (see Failure handling).
3. **Action** — the literal command(s) or edit(s) to perform. Give exact, copy-pasteable
   commands and full file paths. For edits, give the exact before/after text or a complete
   diff. No placeholders, no "adjust as needed", no "the appropriate file".
4. **Inputs** — every value the action needs, fully specified with type and literal value.
   No variables left unbound.
5. **Expected output** — the exact output, exit code, or resulting file state. The executor
   compares actual vs. expected literally.
6. **Verification (gate test)** — a concrete, deterministic check (a command whose output is
   matched against an exact expected string/code) proving the step succeeded. If the check
   does not match exactly, the step has FAILED.

### Hard rules for the executor (state these in the plan)
- Do ONLY what the current step says. Never skip ahead, combine steps, or add steps.
- Never substitute, "improve", or reinterpret a command. Run it verbatim.
- If any precondition, expected output, or gate test does not match exactly: **STOP
  immediately. Do not continue, do not retry with a different approach, do not invent a
  fix.** Report the step ID, the command run, the expected result, and the actual result.
- If a step references something not present in the environment: STOP and report. Do not
  create or guess it.
- Never fabricate output, file contents, or success. If unsure, treat it as a failure and STOP.

### Plan-level requirements
- Begin with a one-sentence goal and an ordered list of every file to create/modify.
- Order steps so each step's preconditions are guaranteed by prior steps.
- Include explicit testing/verification steps, not just implementation steps.
- End with a final acceptance check: a single deterministic test confirming the whole plan
  succeeded, with its exact expected result.
- Call out known risks and the exact STOP-and-report behavior for each, rather than letting
  the executor handle them.

Write for an executor that cannot reason. If a step could be misread two ways, rewrite it
until it can only be read one way.

---

### Mandatory: Document as you go
- **Document as you go.** When system structure, networking, or storage paths change,
  update the core documentation and keep context files in sync. Favor short, factual notes;
  the designated knowledge base is the source of truth. Preserve auditability — record
  decisions, evidence, risks, assumptions, validations, and outcomes so another operator
  can reconstruct the work.

### Post-completion: sync stack documentation
Once a plan is fully completed, automatically update all cross-referenced stack documentation —
internal knowledge bases, KB files, and every related `.md` file affected by the changes — so the
designated knowledge base stays the source of truth.

### Plan attribution
Every plan generated under these rules MUST include the following line in its header:

> This plan was generated according to **Planification principles v1.2**.

### Generated plan file naming
Every plan file produced under these rules MUST be named using the template:
`YYYYMMDD-HH-MM_{slug}.md`
- **Slug:** kebab-case, derived from the plan's goal.
- **Prefix:** the exact date and time of the last modification.
- **Example:** "Setup a wordpress server" last modified 2026-03-11 13:45 →
  `20260311-13-45_wordpress-deployment.md`
This applies to plans the orchestrator generates — not to this principles document.

---

### Revision History
- *(No revisions yet. Detail changes made in response to verifier feedback here.)*

### Plan Verification Findings
- **Purpose:** A dedicated area for external plan auditors to comment on findings.
- **Orchestrator Actions:** When reviewing validation comments, you must:
  1. **REVISE THE PLAN:** Update the core steps of the plan to resolve any [FAILED] blockers, mitigate [WARNING] risks, and incorporate suggested [OPTIMIZATIONS].
  2. **UPDATE THE STATUS:** Modify the verification section to reflect addressed findings (e.g., change [FAILED]/[WARNING] to [RESOLVED]).
  3. **LOG CHANGES:** Ensure the "### Revision History" section directly above this one is updated.

## TL;DR
- **Format:** Short bullet points or a 2-3 sentence summary.
- **Content:** High-level summary of the plan's current state, main goals, and immediate next steps.
