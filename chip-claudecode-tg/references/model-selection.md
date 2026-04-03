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
