# Self-hosted rollout notes

This note is for operators who already run OpenClaw as a self-hosted service and want to ship Claude ACP fixes safely.

It is intentionally generic and does not assume any private host layout.

## 0. Decide whether you are shipping hardened mode or portable mode

Before rollout, decide which promise you are making to the user.

### Hardened mode
- you are willing to patch or replace installed OpenClaw runtime artifacts for stronger Claude cockpit persistence
- you must also own the post-update verification burden, because a future update can overwrite that runtime layer

### Portable mode
- you are not patching installed runtime/system files
- you rely on stock OpenClaw behavior plus honest operator recovery steps
- the cockpit can still be useful, but persistence/recovery is weaker and more manual

Do not blur these two modes in rollout notes.

## 1. Confirm the real runtime path first

Do not assume the active gateway is running from your source checkout.

Confirm:

- the actual service unit
- the actual `ExecStart`
- the actual installed `openclaw` path

Typical examples:

- user unit: `~/.config/systemd/user/openclaw-gateway.service`
- system unit: `/etc/systemd/system/openclaw-gateway.service`
- npm global install: `~/.npm-global/lib/node_modules/openclaw`

If you skip this, it is easy to patch source code and never touch the live runtime.

## 2. Build from a clean tree

Use a clean checkout or worktree that matches the version you intend to ship.

Recommended flow:

1. fetch the branch you trust
2. create a clean worktree
3. run targeted tests
4. run the build

Why:

- your main source tree may already contain unrelated local edits
- rollout should come from a known state

## 3. Backup the installed runtime before replacing artifacts

Before copying a new `dist`, create a full backup of the current one.

Example pattern:

- `dist.bak-YYYYMMDD-HHMMSS`

This gives you a fast rollback path if startup breaks.

## 4. Replace artifacts atomically enough for rollback

For npm-style installs, the practical path is usually:

1. build fresh `dist`
2. backup installed `dist`
3. copy the new `dist` over the installed one
4. restart the gateway service

You may prefer a full package reinstall, but an artifact overlay is often good enough for a private host if:

- you have a clean backup
- you restart immediately after replacement
- you verify health right away

## 5. Verify more than process liveness

After restart, verify at least:

- service status is `active (running)`
- gateway port is listening
- health endpoint returns live status
- plugin backend reinitializes cleanly

For Claude ACP specifically, do one more step:

- run a real turn through a Claude-bound session, not only `/health`

That catches persistent-lane failures that a process check will miss.

## 6. Keep rollout verification layered

Good post-rollout verification order:

1. unit is up
2. port is listening
3. health endpoint is live
4. `acpx` backend becomes ready
5. a real Claude session turn succeeds

If you stop at step 1 or 2, you can still miss the failure that actually matters.

## 7. Know what this rollout does not solve

Shipping the fixes above does not remove:

- Claude account quota limits
- Claude backend outages
- missing CLI login
- broken Telegram permissions
- wrong bot routing

It fixes the OpenClaw side of persistence and recovery. It does not replace operator checks on the Claude side.
