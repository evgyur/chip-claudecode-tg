# Model selection note — Claude cockpit via ACP

## Practical rule
Если backend = `acpx` и agent = `claude`, то источник истины — не человекочитаемый provider/model string, а тот model id, который реально понимает backend.

## Safe defaults
- `opus` — сильный режим
- `sonnet` — дешевле/быстрее
- `haiku` — если нужен максимально лёгкий контур

## Recommended commands
- `/acp model opus`
- `/acp model sonnet`
- `/acp model haiku`

## If the model “changed” but behavior did not
1. `/acp reset`
2. заново `/acp model <id>`
3. отправить обычный prompt
4. только потом судить, применился ли runtime change

## Do not assume
Не считать, что строка вида `anthropic/claude-opus-4-6` автоматически и честно маппится в тот же backend behavior, если runtime реально идёт через `acpx` + Claude CLI.


## Practical warning about long ids
In some live ACP + Claude CLI setups, a long id may be accepted at the control layer but not become the true backend-native runtime value.

That means a command can appear to succeed while the real Claude runtime still behaves as if it is on a different model family.

When in doubt:
1. reset the ACP session
2. set `opus` / `sonnet` / `haiku` directly
3. verify with a fresh ordinary prompt
