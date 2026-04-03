# chip-claudecode-tg

Public OpenClaw skill for setting up a **dedicated Telegram Claude cockpit**:
- separate Telegram supergroup with topics
- separate OpenClaw agent for Claude work
- topic-level ACP Claude routing via `acpx`
- Claude CLI preflight and login checks
- onboarding + runbook for a repeatable operator flow

## What problem this solves
If you want Claude available next to OpenClaw, this skill helps you avoid the messier path of shoving an Anthropic account directly into the main provider layer and then guessing which runtime is actually answering.

The intended result is a cleaner setup:
- Claude lives in its own Telegram chat
- Claude lives in its own runtime/agent lane
- Claude is used through `acpx` + local `claude` CLI
- operator checks make sure the login and runtime path are real

## Who this is for
Use this if you:
- already run OpenClaw and want a **separate Claude cockpit**
- want Telegram topics as the main UX for Claude work
- want Claude routed through **Claude CLI path** instead of a more direct provider wiring
- care about preflight checks, operator clarity, and fewer integration footguns

## What this skill does
This repo gives you:
- the OpenClaw skill definition
- onboarding docs
- runbook docs
- Claude CLI / ACP preflight checklist
- troubleshooting notes for model selection, warmup, and persistent-session recovery
- a public checklist of the OpenClaw behaviors that make Claude ACP persistent mode reliable
- sanitized rollout notes for updating a self-hosted OpenClaw install without leaking local secrets

## What this skill does NOT do
This repo does **not**:
- log into Claude for you
- guarantee Claude CLI is installed on your server
- magically create a Telegram chat without the required operator/runtime permissions
- guarantee every long provider/model id maps cleanly to your live ACP backend
- eliminate all backend-specific issues by itself

It standardizes the architecture and gives you a reproducible operator path.

## Install
Clone the repo and copy the skill folder into your OpenClaw public skills directory.

```bash
git clone https://github.com/evgyur/chip-claudecode-tg.git /tmp/chip-claudecode-tg && \
mkdir -p ~/.openclaw/skills/public && \
cp -R /tmp/chip-claudecode-tg/chip-claudecode-tg ~/.openclaw/skills/public/
```

## Verify installation
Confirm the skill files are in place:

```bash
ls ~/.openclaw/skills/public/chip-claudecode-tg
```

You should see at least:
- `SKILL.md`
- `references/`

## How to use it
Once installed, use the skill when asking OpenClaw to set up or maintain a Telegram Claude cockpit.

Typical intent:
- "Set up a Telegram Claude cockpit using the chip-claudecode-tg skill"
- "Audit this Claude cockpit against chip-claudecode-tg"
- "Use chip-claudecode-tg to provision a new Claude Telegram room"

## Repo contents
- `chip-claudecode-tg/SKILL.md`
- `chip-claudecode-tg/references/onboarding-ru.md`
- `chip-claudecode-tg/references/runbook.md`
- `chip-claudecode-tg/references/preflight-checks.md`
- `chip-claudecode-tg/references/model-selection.md`
- `chip-claudecode-tg/references/quick-test-checklist.md`
- `chip-claudecode-tg/references/manual-review-checklist.md`
- `chip-claudecode-tg/references/openclaw-required-behaviors.md`
- `chip-claudecode-tg/references/recovery-matrix.md`
- `chip-claudecode-tg/references/self-hosted-rollout.md`

## Important model note
For `acpx/claude`, prefer backend-native model ids such as:
- `opus`
- `sonnet`
- `haiku`

Do not assume long provider/model ids are the canonical control surface unless you have confirmed that on your live backend.

## Operational notes
- After a gateway restart, `acpx` may need a short warmup window before Telegram `/acp` commands succeed.
- A persistent ACP Claude session can look dead even when the overall integration is recoverable.
- If a persistent Claude session gets stuck after a restart, reset or heal the session state before rebuilding the whole cockpit.
- Treat the first failed `/acp status` right after restart as a signal to verify warmup, not as final proof that the entire setup is broken.
- A generic `ACP_TURN_FAILED` with `acpx exited with code 1` can be a Claude session or extra-usage cap, not only a broken Telegram binding.
- Stable persistent mode depends on OpenClaw preserving the right `cwd` and `backend`, not replaying stale `model` values, and refusing to resurrect dead ACP identities.

## Success in 5 minutes
A successful setup looks like this:
1. Telegram Claude chat exists and topics are enabled
2. bot is admin
3. separate Claude agent exists
4. `Codex Control` topic is bound to ACP Claude
5. `/acp status` answers
6. `/acp model opus` or `/acp model sonnet` answers
7. an ordinary prompt in `Codex Control` gets a real Claude ACP reply

## License
MIT — see [LICENSE](LICENSE).
