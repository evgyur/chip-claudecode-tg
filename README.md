# chip-claudecode-tg

Public OpenClaw skill for provisioning a Telegram Claude cockpit with:
- a separate Telegram supergroup with topics
- a dedicated OpenClaw agent
- topic-level ACP Claude routing via `acpx`
- Claude CLI preflight and login checks
- onboarding and runbook for repeatable setup

## Why install this
Use this if you want Claude available inside OpenClaw **without pushing your Anthropic account directly into the main OpenClaw provider path**.

The skill is designed around a cleaner operational path:
- separate Telegram Claude cockpit
- separate agent/runtime
- Claude via `acpx` + local `claude` CLI
- same-server login checks
- explicit onboarding and smoke tests

## Install
Clone or copy the `chip-claudecode-tg/` folder into your OpenClaw skills directory.

Example:
```bash
git clone https://github.com/evgyur/chip-claudecode-tg.git /tmp/chip-claudecode-tg && mkdir -p ~/.openclaw/skills/public && cp -R /tmp/chip-claudecode-tg/chip-claudecode-tg ~/.openclaw/skills/public/
```

## Main files
- `chip-claudecode-tg/SKILL.md`
- `chip-claudecode-tg/references/onboarding-ru.md`
- `chip-claudecode-tg/references/runbook.md`
- `chip-claudecode-tg/references/preflight-checks.md`

## Important note
For `acpx/claude`, prefer backend-native model ids such as:
- `opus`
- `sonnet`
- `haiku`

Do not assume long provider/model ids are the canonical control surface unless you have confirmed that on your live backend.

## Operational notes
- After a gateway restart, `acpx` may need a short warmup window before Telegram `/acp` commands succeed.
- For `acpx/claude`, prefer backend-native model ids like `opus`, `sonnet`, and `haiku`.
- Do not assume long ids like `anthropic/claude-opus-4-6` are always the real runtime truth unless you have verified that on your live backend.
- If a persistent ACP Claude session gets stuck after a restart, reset or heal the session state before assuming the whole integration is broken.
