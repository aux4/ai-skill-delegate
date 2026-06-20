# Delegate Skill

Some work doesn't fit in one turn. It's too big, too slow, needs to run later, or could run in parallel — or it genuinely needs a *separate* agent with its own fresh context. This is the method for handing that work off: dispatch it as a background job, watch it, react when it lands, and split or hand off when that's the right shape. You drive every step yourself with `aux4 jobs`, `aux4 cron`, `aux4 poll`, `aux4 queue` (run `aux4 <pkg> <cmd> --help` for exact flags — don't memorize them). This skill is the *judgment*, not a wrapper.

The goal: never block on slow work, never lose a result, never delegate something you could just do.

## When to delegate (and when NOT)

Delegate when the work is:

- **Too slow for one turn** — browsing, multi-step research, a build/test run, a long analysis. Anything past ~30s should not block your turn.
- **Parallel** — the same operation over many items (import 10 records, verify 8 URLs). Run 2–3 at once instead of serially.
- **Scheduled** — must happen later or repeatedly (a reminder, a poll, a recurring check). Schedule it; never `sleep`.
- **A genuinely separate job** — a focused subtask that deserves its own fresh context window, or work that belongs to *another* agent.

Do NOT delegate trivial or quick work. A file read, a kb lookup, a single quick edit, a one-shot command — just do it directly, in this turn. Delegation has overhead (a job, a notification, a context switch); spending it on something you'd finish in two seconds is pure waste. Delegate the heavy thing, not the cheap thing.

If you delegate, tell the user you're on it and keep the job id — you'll need it to monitor and collect the result.

## Dispatch: launch a background job

Run the slow command as a background job and wire a completion hook so the result comes back to you even if your turn has ended:

```
aux4 jobs run --command "<the slow command>" --onComplete "<notify command>"
```

- `--onComplete` runs on any terminal state; `--onSuccess` / `--onFailure` split by exit code. Use them to write the result somewhere you'll see it (a queue message, a kb entry, a todo comment) — close the loop without holding a turn open.
- The job id is returned immediately; `$AUX4_JOB_ID` is set inside the hook. Capture the id.
- Use a consistent `--path` and a `--source` tag if you run many jobs, so `jobs list` stays filterable.

Then tell the user it's running and move on — don't sit and wait.

## Monitor: watch without blocking

Check progress on demand; never busy-wait in your own turn:

```
aux4 jobs status --id <id>      # RUNNING / COMPLETED / FAILED
aux4 jobs output --id <id>      # stdout so far, even mid-run
aux4 jobs tail --id <id>        # follow live output
aux4 jobs list                  # all jobs and their states
```

To **wait for a condition** without sleeping, use `poll` — it re-runs a command until the expected value appears, then returns:

```
aux4 poll --command "aux4 jobs status --id <id>" --expectation "COMPLETED" --failValue "FAILED"
```

For a recurring check (e.g. nudge yourself every 30s until done), schedule it with `cron add --every "30s" --run "..."` instead of looping. Never `sleep`.

## React: handle completion

When a job finishes (you get the notification, or `status` shows a terminal state):

```
aux4 jobs output --id <id>      # read what it produced
```

- **On success** — process the result, save the useful part (kb / file / your answer), then tell the user.
- **On failure** — read the output to diagnose, then **fix it yourself and retry** with a corrected approach. Try a different angle 2–3 times before surfacing it. Only escalate a failure to the user once you've genuinely tried to recover.

Always verify the result is real before you trust it — an errored command is not a success.

## Self-delegation: a fresh context for a focused subtask

You can hand a subtask to a *new instance of yourself* — a clean context window for one concrete goal:

```
aux4 jobs run --command "aux4 ai agent ask '<one specific, concrete goal>'"
```

Rules:

- **One concrete goal per subtask.** "Find the exact product URL for the KALLAX shelf — visit the site, confirm the page loads, return the URL." Not "research 20 products."
- **Don't delegate vague work.** If you can't state a checkable done-condition, the subtask isn't ready — sharpen it first.
- Wire `--onComplete` so the subtask reports back, and give it a stable conversation/output handle if you need to correlate the answer.
- A subtask that's quick and in-scope for *this* context — just do it here. Self-delegate to get isolation or a fresh window, not as a reflex.

## Parallelize and hand off to another agent

**Parallel fan-out.** For N similar items, launch 2–3 jobs at once rather than serially, then collect:

```
aux4 jobs run --command "<work item 1>" --onComplete "..."
aux4 jobs run --command "<work item 2>" --onComplete "..."
aux4 jobs run --command "<work item 3>" --onComplete "..."
```

Track them with `jobs list`, gather each with `jobs output --id <id>`, then merge the results yourself. Keep the fan-out small (2–3 in flight) — more contention rarely helps.

**Hand to another agent.** When work belongs to a different agent, dispatch its task as a job and coordinate the result over a message channel rather than inline:

```
aux4 queue send --name <agent>.inbox ...      # drop a task/result on a queue
aux4 queue receive --name <self>.inbox        # pull replies addressed to you
```

`aux4 channel` / `aux4 queue` are the inter-agent transport — use them so each agent owns its own context and you coordinate through messages, not by cramming everything into one run.

## Clean up

When the work is done, don't leave jobs lying around:

```
aux4 jobs remove --id <id>     # drop one finished job
aux4 jobs clean --olderThan 24h
aux4 jobs kill --id <id>        # stop one that's stuck or no longer needed
```

Cancel scheduled tasks you no longer need with `cron remove`. Leave the workspace clean for the next run.

## Rules

- Delegate the slow / parallel / scheduled / separate-agent work — do the trivial work yourself.
- Dispatch in the background and wire `--onComplete`; never block your turn, never `sleep` (use `poll`/`cron`).
- Capture every job id; you can't monitor or collect what you didn't keep.
- Every subtask gets ONE concrete, checkable goal — no vague delegation.
- On failure, read the output, fix it yourself, and retry before escalating.
- Verify a result is real before trusting it; save the useful part.
- Coordinate multiple agents through `queue`/`channel` messages, not one mega-context.
- Clean up jobs and schedules when done.
- Run `aux4 jobs`/`cron`/`poll`/`queue` yourself — this skill is the method, not a tool that runs for you.
