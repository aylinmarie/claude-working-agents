---
name: pipeline
description: Runs the full engineer -> tester -> reviewer pipeline on a single task in one invocation, instead of the user manually prompting each agent in turn. Use when the user asks to "run the pipeline," "implement and review," "ship this," or otherwise wants a task taken from implementation through to a review verdict without pausing between steps.
---

Run the three pipeline agents defined in `.agents/agents/engineer.md`, `.agents/agents/tester.md`, and `.agents/agents/reviewer.md` in strict sequence, passing each agent's handoff block forward as the next agent's starting context. Full agent behavior and the handoff formats are documented in AGENTS.md — read it if anything below is ambiguous.

## Steps

1. **Invoke the engineer agent** with the user's task description. Capture the `ENGINEER HANDOFF` block from its output in full.

2. **Invoke the tester agent**, passing it the complete `ENGINEER HANDOFF` block as context (not a summary — the literal block). Capture the `TESTER HANDOFF` block from its output in full.

   - If `Recommendation to reviewer: HOLD (...)`, **stop here.** Do not invoke the reviewer. Report the HOLD reason and the regressions/gaps listed in the handoff back to the user, and note that the engineer needs a follow-up task to address them before the pipeline can re-run.

3. **Invoke the reviewer agent**, passing it both the `ENGINEER HANDOFF` and `TESTER HANDOFF` blocks in full. Capture its verdict.

   - If `VERDICT: REQUEST CHANGES`, stop. Report the blocking issues list back to the user exactly as the reviewer wrote it — these become the next engineer task if the user wants to re-run.
   - If `VERDICT: APPROVED`, report that back to the user as the pipeline's final result.

## Rules

- Each agent runs once. Do not retry an agent within a single pipeline run, even if its output looks incomplete — surface the problem to the user instead of guessing at a fix.
- Pass handoff blocks verbatim between agents. Do not paraphrase or compress them — the next agent's instructions expect the exact field names (`Commit:`, `Base branch:`, `Test suite result:`, etc.).
- Never invoke a later stage after an earlier one signals HOLD or REQUEST CHANGES.
- This skill only orchestrates. It does not itself write code, run tests, or review diffs — that's each subagent's job.
