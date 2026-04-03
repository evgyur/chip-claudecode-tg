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
