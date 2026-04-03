---
name: chip-claudecode-tg
description: "Builds or maintains a Telegram-based Claude Code cockpit for OpenClaw: a dedicated supergroup with topics, a bound ACP Claude agent, working /acp commands, Claude CLI login checks, and a clean topic-per-workstream workflow. Use when someone wants a separate Telegram room where ordinary messages go to Claude ACP instead of a generic cockpit agent."
---

# chip-claudecode-tg

Скилл для настройки и обслуживания Telegram-чата, где **один topic = один поток работы с Claude ACP**.

## Когда использовать
Используй этот скилл, когда пользователь хочет:
- отдельный Telegram-чат под Claude Code / Claude CLI
- topics / ветки вместо хаоса в одной личке
- отдельный agent под Telegram Claude cockpit
- рабочие `/acp ...` команды внутри Telegram topics
- отдельный чат для Claude, не смешанный с Codex cockpit
- проверку, что Claude CLI запущен на том же сервере и уже залогинен
- воспроизводимый публичный runbook без приватных привязок

## Что делает скилл
1. Создаёт или обслуживает Telegram supergroup под Claude ACP
2. Включает topics
3. Добавляет Telegram-бота и поднимает его в admin
4. Создаёт базовую структуру тем
5. Создаёт отдельный OpenClaw agent под Claude cockpit
6. Привязывает `Codex Control` topic к persistent ACP Claude runtime
7. Проверяет `acpx` backend, Claude CLI и login state
8. Даёт понятный онбординг для ежедневной работы

## Базовая структура topics
По умолчанию использовать такую схему:
- Inbox / Triage
- Codex Control
- Runtime / Ops
- Server / Infra
- Scratch / Experiments

Если у пользователя уже есть своя проектная структура — адаптировать, а не навязывать эту сетку.

## Правила работы
- **Один topic = один workstream**
- Не мешать несколько независимых задач в одной ветке
- `Codex Control` держать для bind/status/control
- Проектную работу вести в профильных topics
- Для Claude ACP в Telegram лучше использовать backend-native model ids:
  - `opus`
  - `sonnet`
  - `haiku`
- Не использовать как primary ids строки вида `anthropic/claude-opus-4-6`, если реальный backend — `acpx/claude`
- При правках конфига помнить, что массивы вроде `bindings` могут перезаписываться целиком
- Если slash-команды надо отправлять в Telegram программно, они должны идти через user-side path, а не ботом от самого себя

## Быстрый сценарий для пользователя
1. Зайти в `Codex Control`
2. Отправить `/acp status`
3. Отправить `/acp model opus` или `/acp model sonnet`
4. После этого писать уже обычным текстом
5. Для новой большой задачи уходить в отдельный topic

## Что проверить после настройки
1. Чат существует и topics включены
2. Бот — admin
3. Отдельный agent существует в config
4. Binding указывает именно `chat:topic -> ACP Claude agent`
5. `acpx` backend ready
6. Claude CLI доступен на том же сервере
7. Claude CLI уже залогинен под тем же unix-user / service-context
8. `/acp status` отвечает внутри `Codex Control`
9. `/acp model opus` или `/acp model sonnet` отвечает внутри `Codex Control`
10. Обычный prompt в `Codex Control` реально получает ответ от Claude ACP
11. Онбординг не врёт о реальном UX

## References
- [Публичный онбординг на русском](references/onboarding-ru.md)
- [Публичный runbook](references/runbook.md)
- [Preflight: Claude CLI / login / ACP checks](references/preflight-checks.md)
- [Quick test checklist](references/quick-test-checklist.md)
- [Manual review checklist](references/manual-review-checklist.md)
- [Model selection note](references/model-selection.md)

## Output contract
Когда этот скилл используется, вернуть:
1. структура Claude cockpit
2. как пользователь начинает работу
3. какие `/acp` команды нужны ежедневно
4. проходит ли preflight по `acpx` и Claude CLI
5. какой model id использовать для реального backend
6. какие конфиг-риски или footguns остались
