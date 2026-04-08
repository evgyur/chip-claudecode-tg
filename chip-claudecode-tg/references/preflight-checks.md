# Preflight — Claude CLI / login / ACP checks

Перед автоматическим поднятием Telegram Claude cockpit проверь этот минимум.

## 1) `acpx` backend
Проверить, что backend разрешён и готов:
- `acp.enabled = true`
- `acp.backend = acpx`
- `plugins.allow` содержит `acpx`
- entry `plugins.entries.acpx.enabled = true`

Если backend не ready, сначала чинить это, а не Telegram binding.

## 2) Claude CLI установлен на том же сервере
Проверить:
- `which claude`
- `claude --version`

Если CLI отсутствует — установить его в том же runtime/user context, где будет работать OpenClaw.

## 3) Claude CLI залогинен на том же сервере и под тем же user-context
Это критично. Недостаточно быть залогиненным “где-то ещё”.

Проверка:
- открыть shell под тем же unix-user / service user, который реально использует OpenClaw runtime
- запустить `claude`
- убедиться, что CLI не просит login/auth flow и умеет выполнить обычный prompt

## 4) Если логина нет
Нужно сделать это вручную на сервере:
1. зайти под нужным user-context
2. запустить `claude`
3. пройти OAuth / browser login flow
4. после этого выполнить простой smoke prompt

Пример smoke:
```bash
claude -p "Reply exactly: Claude login OK"
```

## 5) Direct ACP smoke до Telegram
Перед Telegram полезно проверить backend напрямую:
```bash
acpx claude exec "Reply exactly: ACP Claude backend OK"
```

Если этот шаг не работает, Telegram routing ещё рано чинить.

## 6) Model ids
Для `acpx/claude` сначала пробовать backend-native ids:
- `opus`
- `sonnet`
- `haiku`

Не считать длинные provider/model ids гарантированно рабочими, пока это не подтверждено на живом backend.


## 7) Warmup after restart
After a gateway restart, `acpx` may report temporarily unavailable even though the backend is still warming up.

Practical rule:
- do not treat the first failed `/acp status` immediately after restart as final proof that the whole setup is broken;
- wait a moment and retry;
- if a persistent session still looks dead after warmup, reset or heal that session state before rebuilding the whole Telegram binding.

## 8) Decide patch policy before install
Before the cockpit is installed, the agent/operator should ask the user to choose one of these modes:

- **hardened mode** — allow a live patch of installed OpenClaw runtime files for stronger persistent Claude recovery
- **portable mode** — do not patch installed runtime/system files; accept a softer recovery path instead

What to confirm explicitly:
- the chosen mode
- that hardened mode can be overwritten by a future OpenClaw/package update
- that portable mode still works, but may need `/acp reset`, warmup retry, or model re-selection more often
