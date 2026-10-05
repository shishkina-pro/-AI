---
id: README
updated: 2026-09-29 15:30 GMT+3
---

# Demonstrator Agent Rules

**Для агента primary-источник — эти `.md` файлы** (и этот README). HTML-страница `/agent/` с вкладками — зеркало для человека; не полагайся на вкладки как на полный контекст.

## Скоуп MCP

Агент пишет контент, структурирует курс (порядок, перенос, копирование), soft-delete в корзину и восстанавливает из корзины. Медиа, виджеты, темы, создание share-ссылок (`preview` / `preview_comments`) — да. Большинство write-tools в MCP помечены безопасными (`destructiveHint: false`).

**Вне MCP (не вызывай и не обходи):** permanent delete проекта (`delete-project`); ссылка «Микрообучение» (`link_type: micro`); приглашения и ACL пространства; suggestions propose/apply (WIP); restore из entity-snapshot / backup ZIP.

## MCP — только `app.longread.agency`

Единственная среда: **`https://app.longread.agency`**.

- MCP: `https://app.longread.agency/mcp/demonstrator/mcp`. Токен уже лежит в конфиге MCP-клиента в IDE (`mcp.json` → `headers.Authorization`, значение после `Bearer `) — не бери его из переменных окружения и не проси «из ЛК». В Cursor сервер `demonstrator-mcp` / `user-demonstrator-mcp`.
- Share, материалы, ссылки человеку — только с хостом `app.longread.agency`.
- Если tool вернул share/`url` с `localhost` или `127.0.0.1` — **не отдавай как есть**: замени host на `https://app.longread.agency`, тот же `token` в query. Проверь, что страница открывается.
- Другие MCP-серверы Демонстратора в конфиге IDE **игнорируй**. Не выбирай их.

## Эталон и сдача

- Если человек дал эталон через `@path` — **только этот файл**. Не подменяй `_compare_extract`, «похожим» dump из чата или соседним черновиком.
- Нет явного пути — спроси один источник. Не собирай текст из нескольких неоговорённых кусков.

### Перед сдачей

1. `get-material` — хвост `content` целый (см. `03-mcp-tools.md`): нет дубля раздела внизу, нет голых `===` / обрывков fence.
2. Открой share на `app.longread.agency` и пробеги глазами: нет сырых `===`, нет `:::/…` в видимом тексте, нет лишнего CTA/кнопок вне ТЗ и эталона, логотип/обложка/картинки грузятся (URL с `/images/…`, не «починенный» руками путь), конец материала совпадает с задуманным. **После колонок и слайдера** отдельно: ряд, не столбик; в карусели одна ориентация кадров. Столбик и разнобой ориентаций в md / `verify.ok` не видны.
3. Share-host — `app.longread.agency`, не loopback.

## СТОП: только эти действия — после прямого «да»

Большинство MCP-действий (писать урок, порядок, темы, первая публикация share, bake после BRIEF — см. `05`) агент делает без отдельного «можно ли?» на каждый шаг.

**Запрещено без прямого подтверждения пользователя** (в MCP у них `destructiveHint: true`):

- **Удаление материала:** `delete-material` (soft-delete в корзину)
- **Удаление ресурса медиатеки:** `delete-resource` (навсегда, без корзины)
- **Просмотровая ссылка (`preview`):** снятие `delete-*-share-link` с `link_type: preview`; обновление `update-material-share-link` (`auto_update`)
- **Ссылка с комментариями (`preview_comments`):** снятие `delete-*-share-link` с `link_type: preview_comments`; обновление снапшота `republish-comments-snapshot`

**Обязательно в вопросе пользователю:** цель → конкретные шаги (tools + id) → риск (что потеряется) → варианты. Жди ответа. Не меняй план втихую.

Сбой MCP (401, timeout) — тоже стоп и вопрос. Не «лечи» пересборкой. То же для `curl`-загрузки картинок: `401` от `/api/resources` — токен не тот или ротирован; стоп и спроси актуальный токен у пользователя. Не ищи токен в переменных окружения шелла.

Bake виджета / insert — по-прежнему только после BRIEF и прямого «да» (см. `05-html-widget-rules.md`), даже если annotation у bake не destructive.

Выбери сценарий ниже.

## Сценарий 1: IDE-агент с MCP (Cursor, Claude Code, VS Code)

Ты можешь писать файлы и вызывать MCP-инструменты.

1. Правила уже в проекте (человек распаковал zip с `/agent/`) или читай raw `.md` с https://app.longread.agency/agent/ — entrypoint `SKILL.md`, затем `README` + файлы из таблицы ниже (`01-syntax` … `06-russian-typography`). Старых имён `03-media` / `04-mcp` / `05-ai-html` нет. Word/PPT на диске — опционально отдельный zip вспомогательных правил (`demonstrator-agent-helpers.zip`), не часть ядра.
2. Токен уже в конфиге MCP-клиента (`mcp.json` → `headers.Authorization`). В Личном кабинете он был сгенерирован один раз при первичной настройке — не запрашивай его заново и не ищи в env.
3. Подпишись на remote MCP:

```json
{
  "mcpServers": {
    "demonstrator-mcp": {
      "url": "https://app.longread.agency/mcp/demonstrator/mcp",
      "headers": {
        "Authorization": "Bearer <токен_из_mcp.json>"
      }
    }
  }
}
```

4. Аргументы tools — из `tools/list` после подключения.

### Happy path: материал с нуля

1. `list-projects` — найди `project_id` (или `create-project`).
2. `create-material` — `title` + `project_id` (контент можно пустым).
3. Собери markdown по `01-syntax.md` / `02-methodology.md`.
4. Запиши тело через **Yjs collab-мост**: `apply-material-point-patch` (`mode: "replace_all"`) или `save-material-content`. Title — через `save-material-content` + `title`. Тема: если у проекта уже есть кастомный пресет (например «Журнал») — `inherit-project-themes` на уроки, overlay пустой; не копируй `contentStyle` на каждый материал при `styleId: default`. Иначе — `apply-material-theme` **полным пресетом** (палитра + стиль; не одни поля стиля — см. `03` § «Стиль ≠ пресет»). Не raw PATCH `content` в БД (открытый редактор откатит). Tool `update-material-content` удалён.
5. После записи: `verify.ok` **и** `verify.integrity.ok`, плюс хвост `get-material` при сомнении (см. `03`). Не сдавай по одному `verify.ok`.
6. **Порядок карточек проекта.** Порядок создания в сетку не переносится: каждый новый корневой материал `create-material` вставляет в **начало** `project_sequence`. Нужен свой порядок — после создания задай его явно: `get-project-sequence` → `patch-project-sequence` (список сверху вниз). `reorder-materials` правит только `sort_order` внутри раздела и сетку проекта не двигает (см. `03-mcp-tools.md` § «Порядок: три разных оси»).
7. **Ссылка на просмотр** (проект / раздел / материал): `recipe-share-link` с `entity`, id и `link_type` (`preview` или `preview_comments`). Ответ содержит канонический `url` на `app.longread.agency`. Первая публикация — это создание ссылки через MCP, не кнопка «Опубликовать» в ЛК. Не вызывай `create-*-share-link` с `refresh_only: true`, если ссылки этого типа ещё нет (404 «не опубликован»). Подробности — `03-mcp-tools.md` § Share.

Медиа с диска — `03-mcp-tools.md`:

- картинка для показа (логотип, обложка): `kind: image`, канон — multipart `curl` на `/api/resources` (`upload-local-media.ps1` — обёртка Windows; на Linux — тот же `curl`, не Python) → `full_url` с `/images/…webp`, затем `insert-media-block` (insert файл не заливает). **Обложка проекта** — лента ~5:1, не кадр 16:9 (см. `01-syntax.md`, CDN-заполнители). Токен для `curl` — тот же Bearer, что в конфиге MCP-клиента в IDE (сервер `demonstrator-mcp` / `user-demonstrator-mcp`); не из переменных окружения шелла. `401` на `/api/resources` — стоп и вопрос пользователю;
- audio/video/**скачиваемый** file: MCP `presign-media-upload` → PUT → `complete-media-upload` (`files/originals/` у file — норма, не «баг»);
- уже HTTPS: `upload-media-from-fs` только с `file_url` и верным `kind`.

Не пиши Python upload. Не ищи helper вне workspace. Не спрашивай способ заливки. Путь/имя файла — ASCII-only; русское имя — после upload. **Не переписывай** сегменты CDN (`originals`/`processed`/`images`) руками. Не ищи токен в переменных окружения шелла.

ИИ-HTML: правь и отлаживай **локальный `.html`** (IDE с MCP) или передавай HTML в `save-widget-draft` без диска (агент без IDE). Свободный HTML агента — **автономен** (цвета литералами, не `--longread-*`); тема в `create-widget` / `save-widget-draft` — полный пресет, не одни `styleId` / `contentStyle` (см. `05`, `03`). `bake-widget`, CDN и `insert-ai-widget-block` — только после прямого вопроса и «да» пользователя. Не пеки на каждой правке. В теле `:::ai_widget` — **CDN `entry_url`** из `archives/` после `ready`; не bare id, не `draft_url`, не свежий `get-widget`. После bake poll **`archives/`**, пока маркер не совпадёт с локальным HTML (IDE) или вернёт `ready` (Agentic). Draft и `get-widget` — не сдача. Финал зависит от окружения: в IDE — вставка `:::ai_widget` в материал; у агента без IDE — **CDN `entry_url` в чат**, а вставка `:::ai_widget` — по отдельному «да». Чат-боты без MCP bake не делают — отдают код. Детали — `05-html-widget-rules.md`.

## Сценарий 2: Чат-бот без MCP (Gemini Web, ChatGPT, NotebookLM, WebUI)

Ты **не** качаешь файлы и **не** кладёшь их в RAG сам — у тебя нет такого доступа. Правила в контексте появляются только если **человек** заранее приложил `.md` (Knowledge / Project files / загрузка в чат).

Если правила уже в контексте — работай как референс:

1. Опирайся на цепочку `01-syntax` → `02-methodology` → `04-css` → `05-html-widget` → `06-russian-typography`. `03-mcp-tools` — только обзор (API не вызываешь).
2. Отдай пользователю **готовый markdown** материала. Он вставит в редактор вручную.
3. Если файлов нет во входе — попроси человека скачать zip с https://app.longread.agency/agent/ и приложить.

### Для человека (как дать боту правила)

На https://app.longread.agency/agent/ нажми **«Скачать .zip с правилами»**. Распакуй и загрузи `.md` в Knowledge / Project files / вложение чата — до просьбы писать материал. Не нужно копировать curl.
### Медиа без MCP

- Не выдумывай URL и не обещай загрузку файлов.
- Либо оставь **CDN-заполнитель** из `01-syntax.md` (`https://cdn.longread.agency/demo/...`).
- Либо **спроси у пользователя** готовый HTTPS-URL (или resource id, если он сам загрузил в ЛК).
- ИИ-HTML без MCP: отдай **готовый** фрагмент `:::ai_widget` … `:::/ai_widget` (и HTML, если нужно); bake пользователь сделает в UI или через агента с MCP. Не ограничивайся одной ссылкой на HTML.

## Файлы

| Файл | Содержание |
|---|---|
| `SKILL.md` | Короткий entrypoint-скилл агента |
| `01-syntax.md` | Синтаксис Markdown и блоков `:::тип` |
| `02-methodology.md` | Какой блок выбрать + anti-patterns |
| `03-mcp-tools.md` | Карта MCP-инструментов и подключение |
| `04-css-generation.md` | CSS-темы (`--longread-*`) |
| `05-html-widget-rules.md` | ИИ-HTML (`:::ai_widget`) |
| `06-russian-typography.md` | Русская типографика (—, «ёлочки», …, NBSP) |

Пакеты на `/agent/`: `demonstrator-agent-rules.zip` (ядро `01`–`06`) и необязательный `demonstrator-agent-helpers.zip` (вспомогательные правила пайплайна — отдельно, не продолжение нумерации).

Старых имён `03-media.md`, `04-mcp.md`, `05-ai-html.md` **нет** — не угадывай их (404).

## Даты обновления

В шапке каждого файла:

```yaml
---
id: …
updated: 2026-08-06 15:40 GMT+3
---
```

Необязательное поле `fork:` — рядом с `updated:`, когда потребитель держит локальные добавления поверх базы:

```yaml
---
id: …
updated: 2026-09-16 17:20 GMT+3
fork: local additions; merge on update, do not overwrite
---
```

- `updated:` — версия **базы** (эталон с `/agent/`).
- `fork:` — файл содержит локальные добавления; при обновлении **сливай** с публикацией, не перезаписывай целиком и не считай «локальный `updated` новее → не трогать».

Сверка:

1. Открой локальный `.md` → поле `updated:` (и `fork:`, если есть).
2. Сравни с **сырым** тем же файлом на `https://app.longread.agency/agent/<файл>.md` (HTTP body целиком / curl / Read). HTML-страница `/agent/` и WebFetch/markdown-render часто **срезают YAML-фронтматтер** — по ним `updated:` «пропадает» ложно; это не значит, что поля нет на сервере.
3. Если локальный `updated` старше и **нет** `fork:` — снова скачай zip с `/agent/` или raw `.md`.
4. Если есть `fork:` — подтяни базу с `/agent/` и **слей** локальные добавления; не затирай находки слепой перезаписью и не игнорируй публикацию из‑за «локальный новее».
5. Не сравнивай по mtime и **не** по размеру файла: сервер отдаёт содержимое в символах, диск — в байтах, размеры несравнимы. Для побайтовой сверки — `Get-FileHash` (Windows) или `sha256sum` локального файла против скачанного raw `.md`.
6. Часовой пояс в `updated:` — **GMT+3** (Москва).
7. Не гоняй curl по всем восьми файлам из‑за «срезанного» frontmatter у одного tool — перечитай один raw `.md`.

Работай **только по источнику** (`/agent/`), не «по памяти». У локальной копии без `updated:` — она вне механизма сверки, эталоном не считать (кроме явного `fork:`: база всё равно с `/agent/`). Не выдумывай «механизмы сверки» и несуществующие файлы/папки: сверка — `updated:` (+ опционально `fork:`) + чтение raw с `/agent/`; при сомнении — хеш raw.
