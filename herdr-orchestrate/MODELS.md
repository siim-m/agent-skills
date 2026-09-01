# Model selection for orchestrated work

Opinionated rankings maintained by the repo owner between runs (a run does not modify the skill package). Higher = better on every axis: a high cost score means the model is *cheap* for the owner (it reflects what the owner actually pays, not list price). Intelligence is how hard a problem the model completes unsupervised. Taste covers UI/UX, code quality, API design, and copy.

| model       | cost | intelligence | taste | native harness |
| ----------- | ---- | ------------ | ----- | -------------- |
| gpt-5.6-sol | 9    | 8            | 6     | Codex CLI (`codex`) |
| opus-5      | 6    | 8            | 8     | Claude Code (`claude --model 'claude-opus-5[1m]'`) |
| fable-5     | 2    | 9            | 9     | Claude Code (`claude --model 'claude-fable-5[1m]'`) |

Run every model in its native harness. Do not proxy one vendor's model through another vendor's harness.

## Defaults by task

- Bulk/mechanical implementation (clear spec, migrations, data plumbing): gpt-5.6-sol at medium.
- Heavy grunt work where taste matters little but volume is large: gpt-5.6-sol at xhigh.
- API design, domain modeling, anything user-facing (UI, copy): fable-5 at high, or taste ≥ 7.
- Code review: two reviewers in parallel, consolidated by the orchestrator: gpt-5.6-sol plus fable-5. Per-ticket rounds run both at high; spec-level rounds run both at xhigh. Sol reproduces findings by executing probes; fable finds structural issues by reading; each catches what the other misses. Budget fallback: gpt-5.6-sol at high alone, run as the protocol's single-reviewer case. There is no rule that a reviewer must be a different lab than the implementer.
- Never Haiku.
- Plain `claude-fable-5` (no suffix) is the short-context variant that auto-compacts mid-task; an implementer may run it when boundary compaction (HERDR-OPS.md) is not worth the babysitting.

## Effort

- Claude Code: launch with `--effort {low|medium|high|xhigh}` (xhigh exists on the Claude 5 models; older ones stop at high).
- Codex CLI: launch with `-c model_reasoning_effort={low|medium|high|xhigh}`.
- Effort above high is rarely worth the wall-clock; reserve it for genuinely hard work and say so when proposing it. The grunt-work xhigh default above is the owner's standing exception — cheap volume, not hard work.

Defaults, not limits: judge the output, not the price tag. Escalating a model costs less than shipping mediocre work.
