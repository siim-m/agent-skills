---
name: implement-review-loop
description: Implement a ticket with TDD, then run fresh-reviewer code-review rounds until converged, and report what was fixed and what was refuted.
disable-model-invocation: true
---

Implement a ticket, then loop code review with fresh reviewers until **converged**: a round changes no code — it returns no findings, or you refute every finding with a written reason. Cap the loop at 8 rounds; if round 8 is not converged, stop and report.

The argument is the ticket: an issue tracker URL or ticket number, or a path to a local markdown file.

## Setup

1. **Ask these in your first reply, before any tool call** — one message, skipping whatever the invocation already answered. Then wait for the answer:
   - Where to work: the current checkout, a new branch, or a worktree.
   - The reviewer: which model, and — when the mechanism can set it — at which reasoning effort. Default: `gpt-6-astra` at `high`. Mechanism: named agents whose frontmatter carries the model and effort (e.g. `gpt-6-astra_high`); if no named agent exists for the model/effort pair, fall back to a dynamic Workflow — its `agent()` calls accept model and effort directly.
2. Read the ticket. Ask about anything that needs clarification.
3. Record the pre-implementation `HEAD` as the **fixed point**. Every review round diffs from it, so each round's reviewers see the full change, not just the latest fixes.

**Settle all of the above before you start implementation.**

## Implement

Implement the ticket. Use /tdd where possible, at pre-agreed seams. Run the project's validation (tests, lint, typecheck) and get it green, then commit — before the first review round.

## Review loop

/code-review owns the round's reviewer structure (which sub-agents, what each axis covers, how to aggregate). Each round:

1. Invoke /code-review with the Skill tool and follow its structure. Invoke it fresh each round, and re-invoke it after any compaction before acting — a summary's paraphrase of the skill is not the skill. Layer these constraints on the reviewers it spawns:
   - Pass the fixed point recorded in Setup — the same one every round.
   - Spawn each reviewer through the mechanism settled in Setup. This overrides any subagent type /code-review names.
   - Every reviewer is **fresh**: no prior involvement in this task.
   - Give each reviewer a compact brief, not conversation history: the ticket reference and enough context to check the change against it (include the parent spec issue if there is one), the change scope (files/modules touched), checks already passing, and — from round 2 on — what earlier rounds found, fixed, and refuted (with reasons), so reviewers hunt new issues instead of relitigating.
   - Ask for findings only (no code changes), ordered by severity within each axis, with file:line references.
2. Judge every finding yourself — reviewer output is advice, not authority:
   - **Fix** findings that are real: bugs, broken contracts, missing tests around changed behavior, violations of project conventions.
   - **Refute** findings that are wrong, already handled, out of scope, or a deliberate tradeoff — and record the reason. Refute a high-severity correctness or security finding only after you check the code path it names.
3. Re-run validation after fixes, then commit the round's fixes (unless the user said not to commit; /code-review diffs committed work, so uncommitted fixes stay visible to the next round as unfixed).
4. If the round changed code, run another round.

## Report

When converged, report to the user:

- What was implemented and how it was validated.
- Rounds run and the reviewer model/effort used.
- Findings fixed.
- Findings refuted, each with its reason.
- Anything still open, if the loop stopped without converging.
