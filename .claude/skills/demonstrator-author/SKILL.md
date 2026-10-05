---
id: SKILL
name: demonstrator-author
description: Создание и правка материалов в Демонстраторе — редакторе курсов с нейросетями. Используй при работе с лонгридами, статьями, блоками :::тип и MCP-инструментами.
updated: 2026-09-29 15:30 GMT+3
---

# Демонстратор — агент

Primary: raw `.md` на https://app.longread.agency/agent/  
Начни с `README.md`. HTML-вкладки — зеркало для человека.

Этот файл — короткий entrypoint-скилл. Полные правила — в `01`–`06` рядом с ним.

Имена файлов ядра: `SKILL.md`, `README.md`, `01-syntax.md`, `02-methodology.md`, `03-mcp-tools.md`, `04-css-generation.md`, `05-html-widget-rules.md`, `06-russian-typography.md`. Файлов `03-media`, `04-mcp`, `05-ai-html` **нет** — не подставляй старые имена (они отдают 404).

## Перед началом

1. Правила уже в контексте (человек положил zip) или открой `README.md` + `01`–`06` с https://app.longread.agency/agent/
2. Работай **только по источнику** (`/agent/`), не «по памяти» и не по локальным копиям без сверки.
3. Сверь поле `updated:` в frontmatter с версией на `/agent/`. Читай **сырой** `.md` целиком (`https://app.longread.agency/agent/<файл>.md` через curl / HTTP body / Read). Не HTML-страницу `/agent/` и не WebFetch/markdown-render — они часто **срезают YAML-фронтматтер**, и тогда `updated:` «пропадает» ложно.
4. Если локальный `updated` старше и нет `fork:` — перечитай raw `.md` с `/agent/` (или попроси человека заново скачать zip). Если есть `fork:` — слей базу с `/agent/` с локальными добавлениями, не перезаписывай и не игнорируй публикацию.
5. У локальной копии **нет** `updated:` — она вне механизма сверки; эталоном её не считать. Если tool «не показал» фронтматтер — это сбой чтения, не отсутствие поля: перечитай raw.
6. Не сравнивай по mtime и не по размеру файла (символы ≠ байты). Смотри `updated:` (+ опционально `fork:`); при сомнении — `Get-FileHash` / `sha256sum` локального и скачанного raw.
7. Не выдумывай «механизмы сверки» и несуществующие файлы/папки. Сверка — `updated:` (+ `fork:`) + чтение raw с `/agent/`. Не гоняй curl по всем восьми файлам «потому что WebFetch срезал шапку» — один raw-файл достаточно.
8. Работай только в текущем workspace. Не лезь в соседние репозитории и не сканируй диск за helper’ами.
9. MCP только `https://app.longread.agency/mcp/demonstrator/mcp`. Share-URL только на `app.longread.agency` (loopback → перепиши host). Другие MCP Демонстратора не трогай.
10. Эталон — только явно указанный `@path`. После сейва — хвост `get-material`. Перед сдачей — share глазами (`README`: «Эталон и сдача»).
11. **«Дай ссылку» / опубликовать для просмотра** → `recipe-share-link` (`entity` + id + `link_type`). Не ходи в ЛК жать «Опубликовать». Не вызывай `create-*-share-link` с `refresh_only: true` на ещё неопубликованном (это 404). Детали — `03-mcp-tools.md` § Share.

## Маршрут

- Happy path и сценарии → `README.md`
- Синтаксис → `01-syntax.md` (в т.ч. таблица `===` по типам блоков; в `:::test` вопрос на строке `=== Вопрос?`)
- Выбор блока + anti-patterns → `02-methodology.md`
- MCP: `https://app.longread.agency/mcp/demonstrator/mcp`; токен уже в конфиге MCP-клиента в IDE (`mcp.json` → `headers.Authorization`), не из env и не «из ЛК». Args из `tools/list` → `03-mcp-tools.md` (тело — Yjs collab; после write — хвост `get-material`). Тема — **пресет** (палитра + стиль), не одни `styleId` / `contentStyle`. Ссылка на просмотр / проект / раздел — `recipe-share-link`, не ручной `refresh_only` и не «открой ЛК → Опубликовать»
- Порядок карточек проекта **не** следует из порядка создания: корневой `create-material` встаёт в начало `project_sequence`, а `reorder-materials` двигает только `sort_order`. Свой порядок — `get-project-sequence` → `patch-project-sequence` (`03-mcp-tools.md` § «Порядок: три разных оси»)
- **Локальная картинка → curl (канон), не MCP.** `file_path` в MCP = путь на удалённом сервере, не диск агента. Канон заливки — multipart **`curl`** на `/api/resources`. `upload-local-media.ps1` — обёртка для Windows; на Linux/macOS — тот же `curl`, не Python (см. `03-mcp-tools.md` § «Картинка — curl / upload-local-media.ps1»). Токен для `curl` — тот же Bearer, что в конфиге MCP-клиента в IDE (сервер `demonstrator-mcp` / `user-demonstrator-mcp`); не ищи его в переменных окружения шелла. `401` от `/api/resources` — стоп и спроси актуальный токен у пользователя. Аудио/видео/файл с диска: `presign-media-upload` → Shell `curl PUT` → `complete-media-upload`. `recipe-upload-and-insert` — для HTTPS `file_url` или server-side `file_path`, не для путей клиента.
- Локальная картинка для показа → `kind: image`, ASCII-путь + `curl` (или `upload-local-media.ps1` на Windows) (`03-mcp-tools.md`), `full_url` как есть (`/images/…webp`). Не `file` / не правка CDN-пути. Insert файл не заливает. Не ищи helper вне workspace
- CSS → `04-css-generation.md` (цвета только `var(--longread-*)`; градиент на весь shared — через `--longread-bg-image` + `--longread-bg-image-size: 100% 100%`, не одним `background` на классе)
- ИИ-HTML → `05-html-widget-rules.md`. Сначала BRIEF + согласие; локальный `.html` (IDE с MCP) или HTML в `save-widget-draft` без диска (агент без IDE); свободный HTML — литералы цветов (автономен), shell конструктора — `--longread-*`; тема виджета — полный пресет. Bake только после «да». Сдача = poll `archives/` + совпадение маркера (IDE) или `ready` + CDN `entry_url` в чат (Agentic), не draft. В теле `:::ai_widget` — CDN `entry_url` из `archives/` после `ready` (не bare id, не `draft_url`). В IDE вставка в материал — часть финала; у агента без IDE вставка `:::ai_widget` — по отдельному «да».
- Русская типографика → `06-russian-typography.md` (тире —, «ёлочки», …, NBSP в материалах)

Без MCP: не качай `.md` в RAG сам — их кладёт человек. Отдай markdown. Медиа — CDN-заполнитель или URL от пользователя. ИИ-HTML — фрагмент с `:::ai_widget`, не голая ссылка. Русский текст — по `06`.

Не дублируй правила из этих файлов здесь.

**СТОП.** `delete-material`, `delete-resource` (навсегда, без корзины), снятие/обновление share `preview` / `preview_comments` (`delete-*-share-link`, `update-material-share-link`, `republish-comments-snapshot`) — только после описания процедуры и прямого «да». Bake виджета — тоже после BRIEF + «да» (`05`). Нет tool в `tools/list` (invite/ACL, `delete-project`, micro-share, suggestions WIP) → спроси пользователя, не обходи. Не меняй план втихую.
