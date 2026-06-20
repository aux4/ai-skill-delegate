# agent/skill-delegate

A native aux4 agent skill that gives an agent a **methodology** for delegating work — running it in the background, in parallel, on a schedule, or handing it to another agent — instead of forcing everything into a single turn. It teaches the agent when to offload versus do work directly, and how to dispatch, monitor, react, self-delegate, parallelize, and clean up.

It is an **instruction skill**: prompt-only, no domain commands, no `run`, and no LLM dependency. It does not wrap `aux4 jobs`. The consuming agent already has `aux4 jobs`, `aux4 cron`, `aux4 poll`, and `aux4 queue`/`channel`, plus the LLM to reason — this skill supplies the discipline the agent applies with its own calls. It depends on `aux4/ai-skill` only (for the shared `ai:skill` profile and the skill contract).

## Installation

```bash
aux4 aux4 pkger install agent/skill-delegate
```

## Quick Start

```bash
aux4 ai skill delegate prompt
```

This prints the methodology. The agent reads it, then drives delegation directly:

```bash
aux4 jobs run --command "aux4 ai agent ask 'verify the KALLAX product URL and return it'" --onComplete "aux4 queue send --name self.inbox ..."
aux4 jobs status --id <id>
aux4 jobs output --id <id>
```

## What the Methodology Covers

- **When to delegate (and when not)** — delegate work that's too slow, parallel, scheduled, or genuinely a separate agent's; do trivial/quick work directly in-turn.
- **Dispatch** — launch a background job with `aux4 jobs run --command ... --onComplete ...`, tell the user you're on it, and capture the job id.
- **Monitor** — `aux4 jobs status`/`output`/`tail`/`list`; `aux4 poll` to wait-until-condition; `aux4 cron` for recurring checks — never `sleep`.
- **React** — read the output; on success process and save, on failure diagnose, fix it yourself, and retry before escalating.
- **Self-delegation** — `aux4 jobs run "aux4 ai agent ask '<one concrete goal>'"` gives a fresh context window; one concrete goal per subtask.
- **Parallelize / hand off** — fan out 2–3 jobs at once and merge results; coordinate other agents via `aux4 queue`/`channel` messages.
- **Clean up** — remove/kill finished or stuck jobs and cancel stale schedules.

Read the full guidance with `aux4 ai skill delegate prompt`.

## Commands

This is an instruction skill — it contributes one command to the shared `ai:skill` profile.

| Command | Description |
|---------|-------------|
| `aux4 ai skill delegate prompt` | Print the delegation methodology (when to delegate, dispatch/monitor/react, self-delegation, parallel + multi-agent) |

## Native Skill Contract

This package conforms to the native aux4 skill contract:

- **scope `agent`, name `skill-delegate`** — depends on `aux4/ai-skill` only; never on `aux4/jobs`, `aux4/ai-agent`, or `aux4/copilot`. It is pure methodology; the agent that uses it already has jobs/cron/poll/queue.
- **`help.text` everywhere** — the required discovery layer (`aux4 ai skill delegate --help`).
- **`man/ai_skill_delegate__prompt.md` and `man/ai_skill__delegate.md`** — the "is this a good fit?" tier.
- **`prompt`** — the methodology itself; this is an instruction skill so `prompt` is its primary surface.
- **No `run`, no domain commands** — running a skill is a runtime concern of `aux4/ai-agent`, and this skill teaches a method rather than performing a deterministic capability.

Verify conformance:

```bash
aux4 ai skill validate delegate
aux4 ai skill list
```

## License

This package is licensed under the Apache-2.0 License.

See [LICENSE](./LICENSE) for details.
