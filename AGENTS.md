# Агент Демонстратора (Longread)

Правила работы с Демонстратором лежат в `.claude/skills/demonstrator-author/`
(распакованный `demonstrator-agent-rules.zip` с https://app.longread.agency/agent/, без изменений).

- Entrypoint: `.claude/skills/demonstrator-author/SKILL.md`, затем `README.md` и `01`–`06` рядом с ним.
- MCP-сервер: `demonstrator-mcp` → `https://app.longread.agency/mcp/demonstrator/mcp` (конфиг: `.mcp.json` / `.cursor/mcp.json`).

## Локальное уточнение для этого репозитория

Токен в репозиторий не коммитится: в `.mcp.json` / `.cursor/mcp.json` заголовок
`Authorization` подставляется из переменной окружения `LONGREAD_API_TOKEN`.
Поэтому «токен из конфига MCP» здесь = значение `$LONGREAD_API_TOKEN`; используй его
же для `curl` на `/api/resources`. Если переменная пуста или сервер ответил `401` —
стоп и спроси у пользователя актуальный токен.

При обновлении правил: скачай свежий zip с `/agent/` и замени файлы в папке скилла целиком
(в них нет локальных правок, `fork:` не используется).
