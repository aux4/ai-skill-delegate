# ai skill delegate

Instruction-skill tests for the `agent/skill-delegate` package. These assume the
package is installed locally so the shared `ai:skill` profile and the
`aux4 ai skill validate`/`list` framework commands can discover it:

```bash
aux4 aux4 releaser install --dir packages/ai/ai-skill-delegate/package --noBuild true
```

## prompt

### should output the delegate methodology heading

```execute
aux4 ai skill delegate prompt
```

```expect:partial
# Delegate Skill
```

### should cover when to delegate

```execute
aux4 ai skill delegate prompt
```

```expect:partial
## When to delegate (and when NOT)
```

### should cover dispatching a background job

```execute
aux4 ai skill delegate prompt
```

```expect:partial
## Dispatch: launch a background job
```

### should cover monitoring

```execute
aux4 ai skill delegate prompt
```

```expect:partial
## Monitor: watch without blocking
```

### should cover reacting to completion

```execute
aux4 ai skill delegate prompt
```

```expect:partial
## React: handle completion
```

### should cover self-delegation

```execute
aux4 ai skill delegate prompt
```

```expect:partial
## Self-delegation: a fresh context for a focused subtask
```

### should cover parallel and multi-agent delegation

```execute
aux4 ai skill delegate prompt
```

```expect:partial
## Parallelize and hand off to another agent
```

### should reference aux4 jobs

```execute
aux4 ai skill delegate prompt
```

```expect:partial
aux4 jobs run
```

### should reference aux4 ai agent ask for self-delegation

```execute
aux4 ai skill delegate prompt
```

```expect:partial
aux4 ai agent ask
```

### should reference poll for wait-until-condition

```execute
aux4 ai skill delegate prompt
```

```expect:partial
aux4 poll
```

### should reference queue for inter-agent coordination

```execute
aux4 ai skill delegate prompt
```

```expect:partial
aux4 queue
```

### should include the rules section

```execute
aux4 ai skill delegate prompt
```

```expect:partial
## Rules
```

## native skill contract

### should be registered under the ai:skill profile

```execute
aux4 ai skill delegate --help
```

```expect:partial
Run work in the background, in parallel, on a schedule
```

### validate should pass

```execute
aux4 ai skill validate delegate
```

```expect:partial
conforms to the native skill contract
```

### list should show delegate

```execute
aux4 ai skill list
```

```expect:partial
delegate
```

### should NOT expose a run command

```execute
aux4 ai skill delegate --help | grep -c "  run" || true
```

```expect
0
```

### should be prompt-only — no domain commands beyond prompt

```execute
aux4 ai skill delegate --help | grep -cE "^  (dispatch|monitor|react|run|schedule|parallel) " || true
```

```expect
0
```

### should NOT delegate to ai agent ask from the .aux4 itself

```execute
grep -c "ai agent ask" ../.aux4 || true
```

```expect
0
```

### should NOT expose a run command in the .aux4

```execute
grep -cE "\"run\"" ../.aux4 || true
```

```expect
0
```
