# OpenClaw behaviors required for reliable Claude ACP persistence

If you want a Telegram Claude cockpit that feels persistent instead of fragile, the Telegram binding is only half of the story.

The other half is the OpenClaw ACP manager behavior.

This is the minimum behavior set that made the real setup stable.

## 1. Fresh retry must not reuse stale ACP identity

If the first turn fails early with something like:

- `ACP error (ACP_TURN_FAILED): acpx exited with code 1`

then the retry path must not blindly reuse the previous persisted ACP session identity.

Required behavior:

- clear persisted ACP identity before the retry
- clear cached runtime handle before the retry
- suppress `resumeSessionId` on the next `ensureSession` attempt

Without this, OpenClaw can loop back into the same dead backend session forever.

## 2. In-place reset must not replay stale model values

Configured binding resets should not restore an old `runtimeOptions.model` value from stale metadata.

Required behavior:

- keep useful runtime options like `cwd`, `runtimeMode`, and timeout-style controls
- do not replay `runtimeOptions.model` during in-place reset unless the operator explicitly set it again

Why this matters:

- a dead Claude session can leave a stale model like `opus` in metadata
- after reset, OpenClaw may silently push that old value back into the runtime
- the operator thinks the session was recreated fresh, but the lane immediately drifts back

## 3. Reset must fall back to configured binding `backend` and `cwd`

Sometimes the stored ACP metadata is partial:

- `agent` still exists
- but `backend` is missing
- or `cwd` is missing

Required behavior:

- resolve the configured binding again during reset
- fall back to configured binding `backend`
- fall back to configured binding `cwd`

Without this, a reset can succeed technically while recreating the lane in the wrong workspace or through the wrong backend.

## 4. Generic `acpx exited with code 1` needs a human error hint

From Telegram, different Claude-side failures can collapse into the same wrapper message:

- `acpx exited with code 1`

Required behavior:

- detect quota-shaped Claude failures
- show a next-step that mentions Claude usage or session cap
- do not always suggest `/acp cancel`

The operator should see something like:

- Claude ACP likely hit a session or extra-usage limit
- check Claude Usage and retry after reset time

This prevents wasted debugging on Telegram bindings when the real problem is account usage.

## 5. Observability needs explicit ACP retry diagnostics

When a cockpit fails live, operators need to know which recovery path OpenClaw actually took.

Required diagnostics:

- `resume_fallback`
- `fresh_retry`
- `status_unhealthy`
- `status_probe_failed`

Useful fields:

- session key
- backend session id
- agent session id
- reason summary

Without these logs, persistent-session incidents look random.

## 6. Persistent mode still needs a warmup rule

Even with the fixes above, a restart can still produce a temporary false negative.

Practical rule:

- do not treat the first failed `/acp status` right after restart as final proof of outage
- allow a short warmup window
- only then decide whether the problem is warmup, stale identity, quota, or real config drift

## Operator acceptance checklist

Your OpenClaw build is in the right shape if all of the following are true:

- early `acpx exited with code 1` retries go fresh
- stale ACP identity is not resurrected after reset
- configured binding reset keeps the correct workspace and backend
- stale `runtimeOptions.model` is not replayed automatically
- quota-like failures produce a human hint
- logs clearly show retry and status-repair decisions

If any of these are missing, the Telegram Claude cockpit may still work, but it will remain noticeably less reliable.
