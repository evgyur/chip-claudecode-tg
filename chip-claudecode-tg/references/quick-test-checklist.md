# Quick test checklist

- [ ] Отдельный Telegram чат под Claude создан
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
- [ ] Онбординг соответствует реальному UX
