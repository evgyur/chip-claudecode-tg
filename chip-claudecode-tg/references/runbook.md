# Публичный runbook — Telegram Claude Cockpit

Цель: поднять и обслуживать Telegram cockpit для Claude ACP без приватных привязок к конкретному пользователю, чату или серверу.

## Что должно получиться
- отдельный Telegram supergroup
- topics включены
- бот добавлен и поднят в admin
- чат привязан к отдельному agent
- `Codex Control` topic маршрутизируется в persistent ACP Claude runtime
- `/acp status` и `/acp model ...` отвечают внутри topics
- Claude CLI жив и залогинен на том же сервере
- обычный prompt в `Codex Control` реально получает ответ от Claude ACP

## Preconditions
Перед началом проверь:
- OpenClaw жив и отвечает
- Telegram transport жив
- `acpx` plugin разрешён и enabled
- `acp.enabled = true`
- `channels.telegram.threadBindings.enabled = true`
- есть доступ на config patch / restart
- Claude CLI установлен на том же сервере
- Claude CLI уже залогинен под тем же runtime-user

## Обязательное install-time решение: hardened vs portable
До любых config/runtime правок агент или оператор должен спросить пользователя, какой режим нужен.

### Вопрос, который надо задать
> Хочешь **hardened mode** с live patch установленного OpenClaw runtime ради более устойчивого Claude cockpit, понимая, что такой патч может слететь при следующем обновлении, или **portable mode** без правки системных файлов, но с более слабым recovery?

### Hardened mode
Использовать, если:
- человеку важнее максимальная устойчивость persistent Claude cockpit прямо сейчас
- допустим live patch установленного runtime
- оператор готов после обновлений проверять, не слетел ли этот слой hardening

### Portable mode
Использовать, если:
- человек не хочет, чтобы install flow правил системные/runtime-файлы
- важнее переносимость и предсказуемость обновлений, чем идеальный self-heal

Что это значит practically:
- Claude cockpit всё ещё можно поднять и использовать
- но нужно честно считать его **best-effort persistent**, а не идеально self-healing
- после restart/warmup/stale-session событий могут понадобиться:
  - повтор `/acp status`
  - `/acp reset`
  - повторная установка модели (`/acp model opus` / `sonnet`)
  - повтор обычного prompt после reset

## Шаги поднятия
### 1) Создать Telegram supergroup
Создай отдельный чат под Claude work.

### 2) Включить topics
Нужен именно forum/topic workflow, иначе весь cockpit деградирует в один длинный поток сообщений.

### 3) Добавить бота и выдать admin
Бот должен видеть group messages и уметь работать с topics.

### 4) Создать отдельный agent
Не привязывай Claude cockpit к `main` или к уже живому Codex cockpit.

Рекомендуемый shape:

```json
{
  "id": "claudecockpit",
  "name": "Claude Cockpit",
  "workspace": "/path/to/agent-workspaces/claude-cockpit",
  "model": "openai-codex/gpt-5.4",
  "runtime": {
    "type": "acp",
    "acp": {
      "agent": "claude",
      "backend": "acpx",
      "mode": "persistent",
      "cwd": "/path/to/workspace"
    }
  }
}
```

### 5) Привязать `Codex Control` topic к ACP binding
Нужен именно topic-level binding, а не просто group-level chat binding, если хочется, чтобы обычный текст в control-topic уходил в Claude ACP.

Пример shape:

```json
{
  "type": "acp",
  "agentId": "claudecockpit",
  "match": {
    "channel": "telegram",
    "accountId": "default",
    "peer": {
      "kind": "group",
      "id": "<chat_id>:topic:<topic_id>"
    }
  },
  "acp": {
    "backend": "acpx",
    "mode": "persistent",
    "cwd": "/path/to/workspace"
  }
}
```

### 6) Проверить `acpx` и Claude CLI
Сначала убедиться, что backend жив и Claude CLI работает локально. См. `preflight-checks.md`.

### 7) Первичная инициализация внутри Telegram
Отправить в `Codex Control`:
- `/acp status`
- `/acp model opus`

Если нужен более дешёвый default:
- `/acp model sonnet`

### 8) Сделать живой smoke
Отправить обычный prompt вроде:
- `Reply with exactly: Claude cockpit OK`

Если ответ пришёл — bind и Claude runtime реально работают.

## Что делать по режимам после smoke

### Если выбран hardened mode
- можно дополнительно применить runtime hardening / live patch, если оператор на это согласен
- обязательно зафиксировать, что этот слой может быть overwritten future updates
- обязательно оставить post-update runbook, как проверять, не слетел ли hardening

### Если выбран portable mode
- не патчить установленный OpenClaw runtime
- оставить систему на стандартном behavior installed build
- в онбординге сразу предупредить пользователя, что:
  - первый status после restart может быть ложным негативом из-за warmup
  - stuck lane лечится через `/acp reset`
  - обычный prompt после reset — часть нормального recovery path
  - это не “идеальный persistent cockpit”, а более мягкий режим без системного patch layer

## Главный footgun
`gateway config.patch` по массивам может перезаписать весь массив, а не merge’ить его.

Критично для:
- `bindings`
- `agents.list`
- других list-based секций

Правило:
- сначала читать текущий config
- потом patch’ить массив осознанно
- после patch проверять, что старые bindings не исчезли

## Второй footgun — неправильный model id
Для `acpx/claude` человек может ожидать ids вроде `anthropic/claude-opus-4-6`, но backend может реально хотеть короткие ids:
- `opus`
- `sonnet`
- `haiku`

Если после `model set` реальный runtime не совпадает с ожиданием, сначала проверить backend-native ids.

## Критерий готовности
Контур считается готовым, если:
- чат существует
- бот admin
- topics существуют
- отдельный agent создан
- topic-level ACP binding указывает на Claude cockpit
- `/acp status` отвечает
- `/acp model opus` или `/acp model sonnet` отвечает
- обычный prompt в `Codex Control` получает ответ от Claude ACP
- онбординг не врёт и соответствует live-поведению
- выбранный режим (`hardened` или `portable`) явно зафиксирован и объяснён оператором


## Troubleshooting footgun — restart warmup and stale persistent sessions
Two different failures can look identical from Telegram:
- the backend is still warming up after restart;
- the persistent Claude session is stuck in a stale error/dead state.

Do not immediately rebuild the whole cockpit.

Safer sequence:
1. confirm `acpx` backend is actually ready
2. retry `/acp status` after warmup
3. if ordinary prompts still fail, reset or heal only the affected Claude session
4. only then consider deeper config surgery
