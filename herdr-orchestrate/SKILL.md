---
name: herdr-orchestrate
description: Orchestrate a multi-worktree implement-and-review run inside Herdr. One implementer agent per issue, optional per-ticket review rounds, a standard full-spec review, and integration as a single spec PR (default), a stacked-PR spine, or independent PRs. Argument is a parent spec issue or a list of issues.
disable-model-invocation: true
---

You are the **orchestrator**: you chart the work, launch and arbitrate the agents, and deliver the report. You write no feature code yourself. Mechanical integration is yours to run directly: merges, rebases, gates, pushes, PR creation. Hand integration to an issue's implementer only when a merge or rebase conflicts, because conflict resolution is judgment work, not mechanics.

## The review protocol

Per-ticket review is opt-in; the spec review is standard (see Chart). Wherever a round runs, these rules apply:

- **Fixed point**: immediately after creating a worktree, record the base **SHA**, not just the branch name. Every round diffs `fixed-point...HEAD`, so each reviewer sees the full change, not just the latest fixes.
- **Dual blind reviewers**: each round runs two reviewers in parallel, one per lab when two harnesses are installed (MODELS.md owns the pairing). Each is a fresh harness conversation with no task history; a reused pane needs a verified context reset (HERDR-OPS.md). With one harness, or under the budget fallback in MODELS.md, run a single reviewer, told it is the sole reviewer; consolidation then collapses to its one report. Reviewers are **blind**: they hear nothing about earlier rounds' fixes. The one exception is **settled points**: refutations you promote at consolidation because their written reason stands on its own, each carried as a one-line claim plus that reason, so a reviewer skips relitigating without seeing fix history. Record the settled set in `state.md`.
- **One-liner prompts**: a reviewer prompt is a short skill invocation, never a brief file. The shape:

  > /code-review the diff {fixed-point}...HEAD in this worktree: ticket #{N} of spec #{M}. Judge overall taste too. Settled: {points}. You are one of two independent reviewers; using your own subagents is fine and encouraged. Findings only, no code changes, file:line refs, your review skill's own report structure, written to {run-dir}/review-{issue}-r{K}-{reviewer}.md, or the exact sentence "No findings."

  (`$code-review` in Codex.) For a spec round, swap the target: name the diff source (the PR's base...HEAD, or the spine range list), name spec #{M}, and write to `review-spec-r{K}-{reviewer}.md`; consolidation and dispositions use `review-spec-r{K}.md` and `disposition-spec-r{K}.md`.
- **Consolidate**: merge the two reports into `review-{issue}-r{K}.md` yourself, preserving the review skill's report structure (separate axes stay separate): dedupe within a section, keep the higher severity, drop a re-raised settled point unless it brings a new argument, and flag direct reviewer disagreements for the implementer to arbitrate in writing.
- **Fix/refute**: the implementer judges every consolidated finding; reviewer output is advice, not authority. It writes `disposition-{issue}-r{K}.md` with one FIX or REFUTE entry per finding: fix what is real; refute what is wrong, already handled, out of scope, or a deliberate tradeoff, with a written reason. A high-severity correctness or security finding is refuted only after checking the code path it names. Re-run gates after fixes, then commit the round.
- **Converged**: a round changes no code. The worktree is clean and either both reviewers wrote "No findings." or every consolidated finding has a REFUTE entry and HEAD equals the round-start HEAD. Any FIX means another round, budget permitting. You arbitrate from the files and the SHAs, not from transcript memory.
- **The round budget ends review, never the work**: the final round's findings still get dispositions, fixes, gates, and a commit; the issue is then treated as converged and proceeds; record `review: exhausted` (vs `converged`) in `state.md`. The report states what one more round would have examined.

## Guard

Run `test "${HERDR_ENV:-}" = 1`. If it fails, say you are not inside Herdr and stop.

## Preflight

Discover, then propose from what exists:

1. Harnesses: which of `claude` and `codex` are installed and logged in, and that they expose the child skills the run invokes (`code-review`, `tdd`).
2. `gh` authenticated; the `gh stack` extension installed when spine mode is in play.
3. The `herdr` skill installed. Load it before issuing Herdr control commands; it and the live CLI own command syntax. Then read [`HERDR-OPS.md`](HERDR-OPS.md) for this run's extra rules, before the first launch, not after something misbehaves.
4. Read [`MODELS.md`](MODELS.md) for model choice; propose only models the installed harnesses actually offer.
5. The `unslop` skill: when installed, apply it to everything the user will read (the pause message, PR bodies, the report).

## Chart

Read the argument. The invocation usually states most of the plan. Apply what it states and chart only what it leaves open:

- **Integration mode.** **Single-PR** is the default for a parent spec with sub-issues: one integration branch you own, every issue merged into it, one draft PR that closes the spec and its tickets. **Spine** builds a stacked-PR chain instead; propose it when the user wants incremental merges or ticket-sized human review. **Independent** sends each branch straight to the default branch; it fits only an edgeless DAG, so a dependency edge forces one of the other modes or drops the dependent from the run.
- **Review dosage.** Per-ticket rounds run only when the invocation asks for them: 1 round when it asks without a number. The **spec review** at the end is standard: 2 rounds against the whole spec diff, and round 2 runs even after a clean round 1. Both knobs are overridable.
- A run charted from a bare issue list has no spec: drop the spec references from prompts and skip the spec review; the report says so.
- Anything unusual: carry your interpretation into the approval pause; ask earlier only when no safe plan can be charted without the answer.

For a parent spec, the issue set is its open sub-issues from the tracker's native sub-issue relation; the parent itself is coordination, not work, and closed children are already done. List the set at the pause so the user confirms membership. Build the dependency DAG over that set from the tracker's **native blocking relations**. If issue bodies also carry a dependency convention, compare; report any disagreement at the pause instead of silently picking a side. The DAG defines the **frontier** of runnable issues.

Then draft the run plan:

- **Order**: spine mode needs one topological order of all issues, the bottom-to-top PR order; prefer fronting any externally time-critical track. Single-PR and independent modes integrate in completion order off the frontier.
- **Wave width**: how many worktrees build concurrently (2-3 typical), or fully sequential.
- **Cast**: per issue, implementer model + effort; the reviewer pair for the run. Choose per MODELS.md and say why for each non-default pick.
- **Permission posture**: the pause asks the user to choose: yolo/bypass on a disposable machine, or auto mode with orchestrator-handled approvals (HERDR-OPS.md's blocked-pane policy).

## The pause

Present the whole plan as one message and **wait for approval**: mode, order or frontier policy, width, cast, review dosage, posture, any DAG disagreements. Create nothing before it. Record approved overrides exactly; if an override conflicts with the guard, repo safety rules, or an unavailable capability, state the conflict and pause again instead of silently approximating.

After approval, create the **run directory** in your scratchpad and tell the user its absolute path. It holds `state.md` plus every brief, review, disposition, and decisions file. `state.md` records: the approved plan (mode, order, DAG, width, cast, review dosage, posture), repo/remote/default branch, the integration branch and PR (single-PR mode) or stack identity (spine) once created, and per issue: implementation fixed-point SHA, worktree, agent and pane ids, branch, phase, round, artifact paths, gate results, educated calls, human interventions, and next action. Phases: PLANNED, IMPLEMENTING, REVIEWING, CONVERGED, JOINING, JOINED, OPEN, BLOCKED.

Update `state.md` on every state change. **After compaction, restart, or a human pane intervention**: re-read this skill and `state.md`, reconcile against live Herdr agent state, `git` HEADs, and GitHub PR state; the file is a checkpoint, not authority over newer live state. Record any discrepancy, then resume from the next-action column.

## Run

The approved wave width is the scheduler bound: at most that many issues IMPLEMENTING or REVIEWING at once; start the next frontier issue only when a slot opens. Integration stays serialized regardless.

An issue becomes runnable when its implementation base is unambiguous. Single-PR mode: all its blockers merged into the integration branch, then base on the branch's current head. Spine: roots base on the default branch; one blocker means that blocker's converged head; several blockers wait until all have joined, then the stack tip. Independent: the default branch. Never pick one of several blockers arbitrarily. Then:

1. `herdr worktree create` off that base, and record the fixed-point SHA.
2. Start the implementer in the worktree pane, in its native harness with the agreed effort.
3. Prompt it inline per [`BRIEF-TEMPLATE.md`](BRIEF-TEMPLATE.md).
4. When it stops: read its completion note from the run directory; verify the implementation is committed, the worktree clean, and the gates green. Without per-ticket review the issue is CONVERGED here. With it, loop per the review protocol: record round-start HEAD, start the fresh reviewer pair, consolidate, hand the implementer the consolidated path with "judge fix/refute into the disposition file, gates, commit", arbitrate from the files and HEADs, checkpoint `state.md`, repeat until converged or the budget ends; either way, record CONVERGED once gates are green and every finding has a disposition.

**Compact before real work**: HERDR-OPS.md owns the compaction policy; the trigger is a findings handoff or a new task to a deep-context claude implementer, never a mechanical instruction.

**The human may jump into any pane.** A human turn in a transcript is authoritative: resync from the pane's current state, never prompt over a live conversation, and log the intervention in the state file.

## Integrate (single-PR mode)

Create the integration branch off the default branch at run start and keep it in its own worktree. Per converged issue, you run:

1. Merge the issue branch into the integration branch. A merge, not a rebase, so the issue branch and its worktree stay valid.
2. On conflict: abort, then have the implementer merge the current integration head into its issue branch in its own worktree, resolve, run gates, commit. When the run reviews per ticket, the resolution gets one review round whose fixed point is the merged-in integration head: the issue's full change as it now stands on the new base. Your merge then replays clean.
3. Full gates on the integration branch, push, record JOINED.

After the first merge lands, open a **draft PR** for the integration branch, marked as closing the spec and its tickets. The PR is the user's window on the train; its body accumulates every "Decisions made without the owner" entry as issues land. Before marking it ready, prune closing keywords for any ticket that ended BLOCKED or excluded, and for the spec itself when any of its work is missing.

A gate failure on an integration result without a Git conflict gets the same treatment as a conflict, in every mode: hand it to the issue's implementer with the failure output; it fixes on its branch, and you re-integrate.

## Join (spine mode)

A converged branch joins only when everything below it in the spine has joined. You run the steps directly:

1. Rebase only the issue's own commits: `git rebase --onto <stack-tip> <fixed-point> <branch>`. Never let Git infer the replay boundary, or it can replay the dependency's commits.
2. On conflict, the issue's implementer resolves. When the run reviews per ticket, the resolution gets one review round whose fixed point is the new base (the stack tip rebased onto): the issue's full change as it now stands there. The implementation fixed point stays untouched in `state.md`; it remains the `--onto` replay boundary.
3. Full gates on the rebased head.
4. Push `--force-with-lease`; open the PR as a draft, based on the previous stack-tip branch (default branch for the bottom PR); `gh stack link` it (the `gh-stack` skill, if installed, owns the mechanics; a stack needs two members, so the bottom PR links when the second joins). Verify from `gh` output that the remote head equals the validated SHA, the PR base is the immediate lower branch, and stack membership and order match the approved spine. Record JOINED; the tip advances.

## Publish (independent mode)

You run the same steps against the default branch: refresh it and, if it moved, `git rebase --onto <default-tip> <fixed-point> <branch>`, with conflicts handed to the implementer as above; full gates; push; open the PR against the default branch; verify remote head and PR base from `gh` output; record the PR and validated SHA, then OPEN.

## Spec review

Single-PR and spine modes end with a spec review; independent mode has no spec and skips this section. When every unblocked issue has joined (no issue joined means nothing to review; report instead), review the whole spec, then fix, without pausing for approval: the PR itself is the user's checkpoint, since nothing merges to the default branch without them.

- The diff: in single-PR mode, the PR's base...HEAD. In spine mode, reviewers run in the stack-tip worktree and get the range list: one `A...B` per contiguous stretch (several when early tickets already merged into a moving default branch), every endpoint a resolvable SHA. A concatenated copy in the run directory is a convenience, not the review target.
- Run the review protocol against that diff: 2 rounds by default, and round 2 runs even when round 1 was clean. It goes in blind, carrying the new settled points.
- Fixes in single-PR mode: hand the consolidated findings to an implementer in a fresh worktree off the integration head, fixed-point discipline as always, and integrate its branch like any issue. In spine mode, same workflow off the stack tip; the fix branch joins as one PR on top of the stack, never amendments to intermediate PRs, which would force rebases and CI runs up the whole train. A finding that breaks an intermediate PR's own mergeability is flagged in the report for push-down instead; that surgery is the user's call.

When the spec-review rounds are done, mark the PR ready for review; in spine mode, every stack PR.

## Stuck issues

Only a genuine wedge blocks an issue: a fork expensive to reverse in both directions, or an unrecoverable failure. Review rounds never block; the budget ends them. A BLOCKED issue and every issue reachable from it in the DAG are excluded; record each excluded issue BLOCKED too, naming the upstream blocker as its cause. Unrelated issues still integrate. When exclusions break a spine, stop at the highest joinable prefix and say so. Leave stuck worktrees intact.

## Educated calls

Mid-run questions the spec did not answer get an **educated call**, not a stall: the implementer picks the answer it would defend (a second opinion from another agent is a legitimate input), records the call, its reasoning, and its reversal path in the issue's decisions file in the run directory, and continues. The calls go into the PR body ("Decisions made without the owner"), and the report aggregates them so the human reviews them in one place.

You hold the same license: when an implementer wedges itself on a stalled judgment, a circling argument, or a blocked approval it cannot see past, make the executive call that unblocks it, log it in the state file, and include it in the report. An executive call resolves judgment, never permissions: a quarantined outward action (an unearned push, a delete outside the worktree, a network post) stays with the human.

## Report

The run terminates when every planned issue is JOINED, OPEN, or BLOCKED, the spec review is done (in the modes that have one), and the report is delivered. A blocked entry names the exact blocker and the next recovery action. The run succeeded only if every issue is JOINED or OPEN; say which of the two you are reporting.

The report lists: the PR, or PRs in spine order with the recommended merge order; review rounds per issue with fixed/refuted counts; the spec-review rounds, their dispositions, and what one more round would have examined; every educated and executive call; every human intervention; every blocked issue with its downstream exclusions.
