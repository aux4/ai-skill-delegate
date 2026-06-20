#### Description

The `delegate` skill gives an agent a methodology for handing work off — running it in the background, in parallel, on a schedule, or to another agent — instead of cramming everything into a single turn. It is contributed to the shared `ai:skill` profile (`aux4 ai skill delegate ...`).

It is an **instruction skill**: it has no domain commands, no `run`, and no LLM dependency. It does **not** wrap `aux4 jobs` — the consuming agent already has `aux4 jobs`, `aux4 cron`, `aux4 poll`, and `aux4 queue`/`channel`, plus the LLM to reason. This skill supplies the *judgment* the agent applies with its own calls: when to delegate vs do it directly, how to dispatch / monitor / react, how to self-delegate a focused subtask, and how to parallelize or coordinate other agents. Its only command is `prompt`, which prints the methodology on demand.

The methodology covers:

- **When to delegate (and when not)** — delegate work that's too slow, parallel, scheduled, or genuinely a separate agent's; do trivial/quick work directly.
- **Dispatch** — launch a background job with `aux4 jobs run --command ... --onComplete ...` and capture the job id.
- **Monitor** — `jobs status` / `output` / `tail` / `list`, and `aux4 poll` to wait-until-condition (never `sleep`).
- **React** — read the output; on success save and report, on failure diagnose, fix it yourself, and retry.
- **Self-delegation** — `aux4 jobs run "aux4 ai agent ask '<one concrete goal>'"` for a fresh context window per subtask.
- **Parallelize / hand off** — fan out 2–3 jobs at once; coordinate other agents via `aux4 queue`/`channel`.
- **Clean up** — remove/kill finished or stuck jobs and cancel stale schedules.

Load this skill when an agent faces work too big, too slow, or too parallel for one turn, or that should run later or on another agent. For quick in-turn work, the agent should just do it — engage this skill when offloading is the right move.

#### Usage

```bash
aux4 ai skill delegate prompt
```

#### Example

```bash
aux4 ai skill delegate prompt
```

```text
# Delegate Skill
...
```
