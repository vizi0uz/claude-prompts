---
name: audit
description: After composing a plan, invoke a general-purpose opus subagent to review it for schema compliance, executor-readiness, and risk. The subagent reports findings only. The orchestrator applies valid fixes directly and discards invalid ones. Supports --severity to scope which findings to act on and --loop to repeat review-and-fix until the plan is clean. Trigger on /audit.
---

# Audit

You are the Orchestrator LLM. After composing a plan, parse $ARGUMENTS for two optional flags, then run the audit per the rules below.

## Flags
- `--severity <level>` — level is one of: critical, major, minor, nit. Acts as a floor: the named level **and every level above it** are in scope. Order, highest to lowest: critical > major > minor > nit. Example: `--severity minor` scopes to minor, major, and critical; nits are ignored. If omitted, all severities are in scope.
- `--loop [N]` — repeat the review-and-fix cycle. N is an optional integer ≥ 1 giving the maximum number of cycles. If `--loop` is given with no N, loop until the plan is clean (subject to the safety cap below). If `--loop` is omitted, run exactly one cycle.

Flags may appear in any order. On an unknown severity value, or an N that is non-integer or < 1, print a one-line usage error and stop without running.

## One cycle
A cycle is one full review-and-fix pass:
1. Invoke a subagent via the Agent tool with `subagent_type` = `general-purpose` and `model` = `opus`. In the prompt, include the full current plan text, ask it to review for schema compliance, executor-readiness, and risk, ask it to tag each finding with a severity (critical/major/minor/nit), and explicitly instruct it to report findings only — it must not rewrite or edit the plan.
2. Wait for findings. Discard any below the in-scope severity floor.
3. For each remaining finding: if valid, apply the fix to the plan yourself; if not, discard it.
4. Save the updated plan.

## Looping
When `--loop` is present, repeat the cycle. After each cycle, print a compact summary: cycle number (e.g. "cycle 2 of 5" or "cycle 2 of ∞"), in-scope findings by severity, what you fixed, and what remains.

Stop at the first condition that applies:
- **Clean** — a cycle yields no valid in-scope findings. Report success.
- **Count reached** — `--loop N` was given and N cycles have run. Report what remains.
- **Safety cap** — `--loop` with no N has run MAX_CYCLES = 10 cycles. Stop and report what remains.
- **No progress** — two consecutive cycles end with the identical set of unresolved in-scope findings. Stop and flag them as stuck.

## Defaults
- MAX_CYCLES = 10 (safety cap for unbounded `--loop`).
- No `--severity`: all severities in scope.
- No `--loop`: one cycle.

Bare `/audit` behaves exactly as before: one cycle, all severities, fix valid findings, save.