# -AI

## Коннектор Longread (MCP `demonstrator-mcp`)

MCP-сервер: `https://app.longread.agency/mcp/demonstrator/mcp` (Streamable HTTP, не `/sse`).
Без токена подключение не сработает.

### Настройка

1. Возьмите API-токен в [личном кабинете](https://app.longread.agency/lk.html).
2. Экспортируйте его в переменную окружения (токен в репозиторий **не коммитим**):

   ```bash
   export LONGREAD_API_TOKEN="ваш_токен"
   ```

3. Запустите клиент из корня репозитория:
   - **Claude Code** — подхватит `.mcp.json` автоматически (подтвердите сервер при первом запуске, проверить: `/mcp`).
   - **Cursor** — подхватит `.cursor/mcp.json`.
   - **VS Code / другие клиенты** — используйте тот же URL и заголовок
     `Authorization: Bearer <токен>`.

Альтернатива для Claude Code без файла:

```bash
claude mcp add --transport http demonstrator-mcp \
  https://app.longread.agency/mcp/demonstrator/mcp \
  --header "Authorization: Bearer $LONGREAD_API_TOKEN"
```

### Правила / скиллы

Содержимое zip-архива от Longread распакуйте в правила проекта
(`.cursor/rules`, `AGENTS.md` или `.claude/skills` — как принято у вас).
