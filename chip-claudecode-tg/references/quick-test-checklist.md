# Quick test checklist

- [ ] Отдельный Telegram чат под Claude создан
- [ ] Пользователь явно выбрал install mode: `hardened` или `portable`
- [ ] Topics включены
- [ ] Бот находится в чате как admin
- [ ] Отдельный agent существует в config
- [ ] Topic-level binding указывает `chat:topic -> ACP Claude agent`
- [ ] `acpx` backend ready
- [ ] Claude CLI доступен на том же сервере
- [ ] Claude CLI залогинен в том же runtime/user context
- [ ] `/acp status` отвечает внутри `Codex Control`
- [ ] `/acp model opus` или `/acp model sonnet` отвечает внутри `Codex Control`
- [ ] Обычный prompt в `Codex Control` получает ответ от Claude ACP
- [ ] Для portable mode отдельно проверен recovery path: `/acp reset` -> `/acp model ...` -> обычный prompt
- [ ] Онбординг соответствует реальному UX
