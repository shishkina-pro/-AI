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

Правила из `demonstrator-agent-rules.zip` распакованы в
`.claude/skills/demonstrator-author/` — Claude Code подхватывает их как скилл
`demonstrator-author`. `AGENTS.md` (его читают Cursor и Claude Code через `CLAUDE.md`)
указывает на эти правила. Обновление — скачать свежий zip с
https://app.longread.agency/agent/ и заменить файлы в папке.
