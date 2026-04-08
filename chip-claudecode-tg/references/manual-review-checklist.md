# Manual review checklist

- [ ] Скилл не содержит приватных chat ids, user ids, токенов или секретных путей
- [ ] Скилл не навязывает live patch системных файлов без явного выбора пользователя
- [ ] Онбординг отделяет Telegram routing, ACP backend и Claude CLI login layer
- [ ] Явно сказано, что Claude CLI должен быть залогинен на том же сервере и под тем же user-context
- [ ] Явно сказано, что для `acpx/claude` безопаснее использовать backend-native model ids (`opus`, `sonnet`, `haiku`)
- [ ] Не обещается магическая auto-login автоматизация без участия пользователя
- [ ] Не обещается merge-safe `config.patch` по массивам
- [ ] Topic-level binding описан точнее, чем group-level bind
- [ ] Runbook и onboarding не противоречат друг другу
- [ ] Есть честное различие между `hardened` и `portable` режимами
- [ ] В portable mode не обещается идеальный self-heal; описан нормальный degraded recovery path
