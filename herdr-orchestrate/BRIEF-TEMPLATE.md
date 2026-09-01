# Implementer prompt template

Prompt the implementer with this directly, inline; it is a prompt, not a file the agent fetches.

The ticket and spec are the single source of truth for scope and testable surfaces; when they conflict, the spec wins; when a ticket is unclear or names nothing testable, fix the ticket (a comment is enough, recorded as an educated call) before launch instead of restating scope in the prompt, so implementer and reviewer read the same words. The prompt carries only what the tracker and the repo cannot: run state and the orchestration contract. Repo facts (gates, install, commit style) live in the repo's agent docs, which the harness loads on its own. Padding dilutes; the prompt is the implementer's whole world.

```markdown
# Task: {issue title} (#{issue number}, part of spec #{M} when the run has one)

Branch `{branch}`, based on `{base branch}` ({why this base: default branch | the run's integration branch | PR #N of this stack}). Treat the base branch's content as given.

## Overlap warnings

{Only when another live branch or a recently merged PR touches the same files: name it, state which side is authoritative for what, and what must stay identical so merges dedupe.}

## Orchestration contract

(Always included.)

- Implement with /tdd ($tdd in Codex) against the surfaces the ticket names. Do not invent seams; do not manufacture tests for pure renames. Pin identities, not labels: assert stable identifiers, not display text.
- Subagents are explicitly authorized and encouraged: exploration, second opinions, parallel legwork.
- All repo gates green before you declare done.
- Never push or open a PR on your own, and merge or rebase only when the orchestrator explicitly instructs it. The orchestrator owns integration and runs it directly; you will be called back only for merge or rebase conflicts, integration gate failures, review findings, or new work.
- If review findings arrive, judge every finding FIX or REFUTE: one entry per finding with a written reason, in the disposition file the orchestrator names. Fix what is real, re-run gates, commit, and wait.
- A question the spec does not answer gets an educated call: pick the answer you would defend (a second opinion from another agent is fair input), record the call + reasoning + reversal path in `{decisions file path}`, and continue. These entries become the PR body section "Decisions made without the owner"; no open-question sections, state decisions.
- When your work is done: write a completion note to `{run-dir}/done-{issue}.md` (what you built, gate results, anything the orchestrator needs), then stop with a clean worktree and wait.

```
