#### Description

The `prompt` command prints the delegation methodology — the full guidance an agent reads on demand before handing work off. It explains when to delegate versus do work directly (delegate the slow / parallel / scheduled / separate-agent work; do trivial work in-turn), how to dispatch a background job with `aux4 jobs run` and an `--onComplete` hook, how to monitor without blocking (`jobs status`/`output`/`tail` and `aux4 poll` to wait for a condition rather than `sleep`), how to react to completion (read the output, save success, diagnose and self-retry failures), how to self-delegate a focused subtask with `aux4 ai agent ask` for a fresh context window, how to parallelize a fan-out and coordinate other agents over `aux4 queue`/`channel`, and how to clean up jobs and schedules afterward.

This command reads `instructions/prompt.md` from the package directory and writes it to stdout. Because `delegate` is an **instruction skill**, this is its primary surface — there are no domain commands. The agent applies the methodology with its own `aux4 jobs` / `cron` / `poll` / `queue` calls; the prompt does not run jobs or call an LLM itself.

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

Some work doesn't fit in one turn. It's too big, too slow, needs to run later ...
```
