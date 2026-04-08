# Recovery matrix — warmup, stale identity, quota, and model drift

Several Claude ACP failures look identical from Telegram.

This matrix helps you separate them before you start editing config.

## Case 1. Restart warmup

Symptoms:

- right after gateway restart, `/acp status` fails once
- a second attempt soon after may succeed
- logs show startup or plugin-local install activity

What it usually means:

- `acpx` backend is still warming up
- the Telegram path is not necessarily broken

What to do:

1. wait a short moment
2. retry `/acp status`
3. retry a normal prompt
4. only then escalate

What not to do:

- do not rebuild the whole cockpit immediately

## Case 2. Stale persistent ACP identity

Symptoms:

- persistent topic used to work
- after restart or failure, ordinary prompts keep failing
- retries look like they are hitting the same dead lane
- logs suggest dead or no-session status

What it usually means:

- OpenClaw is still trying to resume a dead ACP session identity

What to do:

1. reset the affected ACP session
2. verify the next retry goes fresh
3. verify the new session persists fresh ids

What OpenClaw must do correctly:

- clear persisted identity before fresh retry
- avoid passing stale `resumeSessionId`

## Case 3. Quota or session-cap failure

Symptoms:

- Telegram shows `ACP_TURN_FAILED`
- message often collapses to `acpx exited with code 1`
- Claude account usage page shows current session cap hit or extra usage exhausted

What it usually means:

- Claude backend refused the turn because of usage limits

What to do:

1. check Claude usage
2. wait for reset time if needed
3. retry after reset

What not to do:

- do not keep debugging Telegram binding or topic routing first

## Case 4. Model drift after reset

Symptoms:

- operator expects fresh default behavior
- after reset, the lane behaves as if an old model is still selected
- metadata still contains a stale `runtimeOptions.model`

What it usually means:

- in-place reset replayed an old runtime model value

What to do:

1. inspect session metadata
2. remove or ignore stale model replay during reset
3. reapply the desired model explicitly

Safer operator sequence:

1. `/acp reset`
2. `/acp model opus` or `/acp model sonnet`
3. send a fresh ordinary prompt

## Case 5. Partial metadata continuity failure

Symptoms:

- reset technically succeeds
- but the lane restarts in the wrong workspace
- or it comes back through the wrong backend

What it usually means:

- stored ACP metadata lost `cwd` or `backend`
- reset fell back to generic defaults instead of configured binding values

What to do:

1. inspect configured binding
2. inspect stored ACP metadata
3. ensure reset falls back to binding `cwd` and `backend`

## Fast triage order

When a Claude cockpit topic stops answering, use this order:

1. check if this is immediately after restart
2. check Claude usage or session cap
3. check whether the lane is reusing a stale identity
4. check for model drift
5. check for lost `cwd` or `backend`
6. only then do broader config surgery

This order avoids a lot of wasted work.

## Mode note: hardened vs portable

If the operator chose **hardened mode**:
- stronger self-heal is expected
- stale-session recovery should be more automatic
- but the runtime patch layer may be overwritten by updates

If the operator chose **portable mode**:
- expect a more manual recovery path
- `/acp reset` + model re-selection + a second ordinary prompt is a normal fix, not necessarily evidence of a broken install
- do not oversell this as perfect persistence; it is a safer no-system-patch mode with more operator-visible recovery
