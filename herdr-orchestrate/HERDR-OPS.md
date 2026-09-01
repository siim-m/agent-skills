# Orchestration rules on top of the herdr skill

Policy layered on the installed `herdr` skill. Load that skill first; it and the live CLI own command syntax, lifecycle states, waiting, reading, and pane control. This file owns only what they do not: run artifacts, reviewer isolation, compaction, approvals, and harness quirks. When anything here disagrees with the live CLI or harness, follow what you observe, note the discrepancy in the report, and propose the file edit to the human; a run does not modify the skill package.

## Durable artifacts

Transcripts scroll off and compact away; the run-directory files (see the skill) are what arbitration and resumption stand on. Implementers end every task with a completion note in the run directory (BRIEF-TEMPLATE.md), so read results from files, not scrollback; when any other pane's output is unrecoverable, ask that agent to write its answer to a run-directory file and record the path in `state.md`.

## Harness quirks

- Claude args after `--`: `--model <model> --effort <level>` plus the agreed posture: `--permission-mode bypassPermissions` (yolo) or `--permission-mode acceptEdits` (auto, orchestrator handles the remaining prompts). If a start is blocked by a permission classifier, retry with a milder permission flag.
- Pass Claude model IDs exactly as configured, including context-window suffixes: `--model claude-fable-5` overrides a `claude-fable-5[1m]` settings default and silently shrinks the window. Quote the brackets.
- Codex args after `--`: `-c model_reasoning_effort=<level>` plus the agreed posture: `-c approval_policy=never -c sandbox_mode=danger-full-access` (yolo) or `-c approval_policy=on-request` (auto). Codex has no effort flag; the `-c` config is the way.
- Codex invokes skills with a `$` prefix (`$code-review`, `$tdd`), not `/`. The convention has changed before; if `$` misfires, read the pane for what Codex currently accepts.
- Some harness environments hold back subagent use unless the prompt authorizes it. Every brief and reviewer prompt authorizes subagents explicitly, and reviewer independence is phrased as "one of two independent reviewers", a wording that keeps subagents available.
- Send prompts inline; hand anything long (a consolidated findings file, an assembled diff) as a file path the agent reads.

## Compaction

Codex implementers are never compacted; Codex holds up well at depth. Claude implementers default to the long-context model variant (MODELS.md documents the short-context fallback) and compact at work boundaries: compact a deep-context agent before handing it a findings file or a new task, and send mechanical instructions as-is. Boundary compaction loses almost nothing because the durable state lives in the run directory and the committed diff. An agent mid-compaction reports `working`; wait for idle before prompting, or the prompt queues behind the compaction.

## Timeouts are not transitions

A wait or a prompt's `--wait` timing out on long work is normal, not failure. Check the agent's live state; re-arm the wait while it is `working`. For your own condition waits, use the harness's monitor facility rather than background sleep loops; long-lived loops get reaped. Checkpoint every observed transition in `state.md`.

## Fresh reviewer per round

A reviewer pane may be reused; the reviewer *session* may not. Start a new conversation via the harness's own reset (in Codex, `/new`), confirm from the pane that the context actually reset, then send the round's one-liner prompt (contents per the skill's review protocol). Renaming the agent afterward is bookkeeping for your state table, not evidence of freshness.

## Blocked panes (approval policy)

When an agent goes `blocked`, read the exact visible prompt before acting; never assume which key means what:

- Reads of run-directory files, ordinary dev commands, test runs inside the worktree: approve, preferring a "don't ask again this session" option when the prompt offers one.
- Destructive or outward-facing actions (deletes outside the worktree, pushes not yet earned by an integration instruction, network posts): leave blocked, log it, surface it to the human. Executive calls do not cover these.
