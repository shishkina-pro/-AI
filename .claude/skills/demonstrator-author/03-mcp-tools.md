---
id: 03-mcp-tools
updated: 2026-09-29 15:30 GMT+3
---

# MCP-инструменты Демонстратора

Карта инструментов (имя → зачем). Полные схемы параметров **не** дублируются здесь.

## Параметры инструментов

После подключения MCP клиент вызывает `tools/list`. Сервер отдаёт канонические `inputSchema` (required, типы, enums, лимиты).

- Бери аргументы **только** из `tools/list` / подсказок клиента — не из этой страницы и не из догадок.
- Этот файл — ориентир «какой tool выбрать». Контракт вызова — у MCP.
- Без токена схемы недоступны (`401`).
- Soft-delete **материала** (`delete-material`), безвозвратное удаление **ресурса** (`delete-resource`), снятие/обновление share `preview` / `preview_comments` — **только** после описания процедуры и прямого подтверждения (см. `README.md`, блок СТОП). Остальные write-tools — безопасные (`destructiveHint: false`): порядок, темы, запись тела, первая публикация share — без отдельного «да» на каждый шаг.
- Нет нужного tool в `tools/list` → **стоп**, спроси пользователя. Не обходи дырку delete+create / массовой перезаписью.
- **Вне MCP (не ищи в `tools/list`):** `delete-project`, `delete-space`, массовая чистка медиатеки «Удалить лишние», ссылка «Микрообучение» (`link_type: micro`), invite/ACL пространства, suggestions propose/apply (WIP), restore из entity-snapshot / backup ZIP.

## Подключение к MCP

Подпишись на remote MCP-шлюз на сервере Демонстратора. Клонировать репозиторий и запускать `node` локально не нужно.

Токен уже лежит в конфиге MCP-клиента в IDE (`mcp.json` → `headers.Authorization`, значение после `Bearer `). В Личном кабинете он генерируется один раз при первичной настройке — не запрашивай его заново и не ищи в переменных окружения.

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

Эндпоинты шлюза:

| Метод | URL | Назначение |
|---|---|---|
| `GET` | `https://app.longread.agency/mcp/demonstrator/health` | Проверка живости |
| `GET` | `https://app.longread.agency/mcp/demonstrator/sse` | SSE (keepalive; Cursor сюда `url` не ставь) |
| `POST` | `https://app.longread.agency/mcp/demonstrator/mcp` | JSON-RPC / streamable HTTP — **это URL для IDE** |

- В конфиге Cursor / Claude Code в поле `url` указывай **`…/mcp`**, не `…/sse`.
- Заголовок `Authorization: Bearer <токен>` обязателен на `POST /mcp`.
- Без токена — `401`.
- Токен привязан к аккаунту: права и квоты — как в UI Демонстратора.
- MCP и публичный сайт — только `app.longread.agency`. Другие шлюзы Демонстратора не используй.
- Share `url` отдавай с хостом `https://app.longread.agency`. Если в ответе `localhost` / `127.0.0.1` — подставь этот host (token тот же) и проверь ссылку. Loopback человеку не отдавай.

## Happy path: материал с нуля

1. `list-projects` → `project_id` (или `create-project`).
2. `create-material` — `title` + `project_id`.
3. Markdown по `01-syntax.md` / `02-methodology.md`.
4. Запись тела — **только через Yjs collab-мост**, не прямым PATCH в БД:
   - полное тело: `apply-material-point-patch` с `mode: "replace_all"` + `content`  
     или `save-material-content` (тот же `POST /api/collab/materials/:id/patch`);
   - точечно: `find_replace_once` (якорь по тексту; **можно при открытом редакторе** — live Y.Text, без wipe файла);
   - `range_patch` (offsets) — **только при закрытой вкладке**; при открытом редакторе API вернёт 409 `point_patch_blocked_editor_open`;
   - title вместе с телом: `save-material-content` + `title`; theme — `apply-material-theme`.
   - **Переименование без записи тела** (например, снять суффикс «(копия)» после `duplicate-material`): `rename-material` (`material_id` + `title`). Контент не трогается.
5. Проверить `verify.ok` **и** `verify.integrity.ok` / `get-material`. `source`: `yjs_room`, `yjs_room_point` или `db_fallback`. Не сдавай по одному `verify.ok`.

### Happy path: «дай ссылку»

1. `recipe-share-link` — `entity` (`project` \| `section` \| `material`) + соответствующий id + `link_type` (`preview` \| `preview_comments`).
2. Отдай человеку `url` из ответа (хост `app.longread.agency`).
3. **Не** проси нажать «Опубликовать» в ЛК: первая публикация = создание ссылки через этот recipe (или `create-*-share-link` **без** `refresh_only`).
4. **Не** ставь `refresh_only: true` на примитивах, пока ссылки этого типа нет — будет 404 «не опубликован». Recipe сам выбирает флаг.

**Запрещено** писать markdown через raw `PATCH /api/materials` с полем `content` (tool `update-material-content` удалён): открытый редактор перезапишет старым Yjs-документом.

Полный маршрут и сценарий без MCP — в `README.md`.

## Каталог (обзор)

Ниже — имя и краткое действие. Схемы полей — в `tools/list`.

## Проекты и пространства

| Инструмент | Действие |
|---|---|
| `list-projects` | Получить список проектов |
| `create-project` | Создать проект |
| `update-project` | Обновить проект |
| `list-spaces` | Получить список пространств |
| `get-space` | Получить пространство |
| `update-space` | Переименовать пространство |
| `list-space-members` | Список участников (только чтение) |

Создание пространства, приглашения, смена состава/ролей, permanent `delete-project` — только через UI, не через MCP.

## Структура

| Инструмент | Действие |
|---|---|
| `list-sections` | Список разделов (**с `sort_order`**, среди разделов) |
| `create-section` | Создать раздел |
| `update-section` | Обновить раздел (в т.ч. одиночный `sort_order`) |
| `reorder-sections` | Порядок **только среди разделов** (`sort_order`) |
| `delete-section` | Удалить раздел (soft delete) |
| `list-materials` | Список материалов (**с `sort_order`**) |
| `search-materials` | Полнотекстовый поиск по материалам |
| `create-material` | Создать материал (корневой встаёт в **начало** `project_sequence`) |
| `duplicate-material` | Клонировать материал (суффикс «(копия)») |
| `rename-material` | Переименовать материал (только title, контент не меняется) |
| `update-material-placement` | Переместить материал / один `sort_order` |
| `reorder-materials` | Порядок уроков **внутри раздела** (или среди корневых) |
| `get-project-sequence` | Смешанный порядок карточек проекта: раздел ↔ корневой прототип |
| `patch-project-sequence` | Задать смешанный порядок (как DnD в ЛК) |
| `delete-material` | Удалить материал (soft delete) |

### Порядок: три разных оси (не путать)

| Что меняешь | Tool | Где видно |
|---|---|---|
| Раздел ↔ корневой прототип вперемешку | **`get-project-sequence` / `patch-project-sequence`** | сетка карточек проекта в ЛК |
| Порядок разделов между собой | `reorder-sections` (`sort_order`) | среди разделов |
| Порядок уроков внутри раздела | `reorder-materials` + `section_id` | внутри раздела |

**Порядок создания материалов ничего не задаёт.** `create-material` без `section_id` вставляет новый материал в **начало** `project_sequence`, а не в конец: серия созданий 1 → 2 → 3 → 4 даст в сетке карточек обратный порядок (4, 3, 2, 1). `sort_order`, который пишет `reorder-materials`, в этой сетке **не участвует** — он про уроки внутри раздела и про дерево.

Нужен конкретный порядок карточек — задай его явно после создания всех материалов: `get-project-sequence` → `patch-project-sequence` списком сверху вниз. Не рассчитывай, что «создал по порядку — так и встанет».

**Можно** чередовать: раздел → прототип → раздел → прототип. Это штатный `project_sequence`, не запрет продукта.

Стартовый/пустой sequence (миграция + UI-fallback) часто выглядит как «сначала все разделы, потом корневые» — **дефолт, не закон**. Не сочиняй «разделы всегда сверху» / «без разделов только плоский список».

Дерево слева sequence почти не отражает: смешанный список — в сетке карточек.

`patch-project-sequence`: `items: [{ "item_type": "section"|"material", "item_id": N }, …]` сверху вниз. В `items` только **корневые** материалы (без `section_id`). Перед вызовом — подтверждение пользователя (см. СТОП в `README.md`).

`reorder-sections` / `reorder-materials` **не** умеют смешивать типы — для этого только sequence.

## Медиа

| Инструмент | Действие |
|---|---|
| `list-resources` | Список ресурсов (фильтр: project_id, material_id, q) |
| `presign-media-upload` | Init upload audio/video/file/web_archive → `resource_id` + `upload_url` (PUT с машины клиента) |
| `complete-media-upload` | Complete после PUT → `id`, `status`, `full_url` (CDN), `markdown_snippet` |
| `upload-media-from-fs` | Загрузка по `file_url` (HTTPS) или `file_path` **уже на сервере MCP** — не Windows-path клиента. Локальные image — **не сюда** (см. «Картинка — curl / upload-local-media.ps1») |
| `get-resource-status` | Статус обработки медиа (poll до ready / failed) |
| `insert-media-block` | Вставить медиа-блок; `display_name` / `name` → имя в медиатеке |
| `rename-resource` | Переименовать ресурс |
| `delete-resource` | Удалить ресурс навсегда (без корзины; только владелец файла или управляющий проектом) |

Удаление ресурса безвозвратно: корзины у медиатеки нет, ссылки на файл в материалах перестают открываться. Перед вызовом `delete-resource` — подтверждение пользователя. Если файл просто не нужен в библиотеке, но может понадобиться, — не удаляй.

Массовая чистка «Удалить лишние» (`POST /api/resources/delete-unused`) в MCP не выведена — только UI.

Upload и insert: осмысленный `filename` / `display_name`. «Без названия» в медиатеке — брак; сразу `rename-resource`.

### Kind и CDN URL — не путать

В markdown / обложку / логотип ставь **только `full_url` (или `url`) из ответа upload / `list-resources` / `get-resource-status`**. Не собирай CDN-путь руками и **не «чини»** подстроки в URL.

| Назначение | `kind` | Как залить | Типичный `full_url` |
|---|---|---|---|
| Картинка для показа (логотип, обложка, иллюстрация, `![…](…)`) | `image` | `upload-local-media.ps1` (канон, обёртка curl) или `curl` multipart; по HTTPS — `upload-media-from-fs` с `kind: image` | `https://cdn.longread.agency/images/{hash}.webp` (растр) или `…/{hash}.svg` (SVG) |
| Скачиваемый файл (PDF, ZIP, DOCX…) | `file` | presign → PUT → complete | `…/files/originals/{id}.{ext}` — так и должно быть |
| Аудио / видео после `ready` | `audio` / `video` | presign → … | `…/audio/processed/…` или `…/video/processed/…` |
| ИИ-HTML | `web_archive` / bake | см. виджеты | `…/archives/{id}/…` |

**Частая ошибка:** логотип/обложку залить как `file` (получится `files/originals/…`), потом «исправить» на выдуманный `files/processed/…`. Пути `files/processed/` **нет**. Для показа — перезалей как **`image`** и возьми новый `full_url` с `/images/…webp`.

Картинка-image: jpeg / png / webp / gif / svg. SVG принимается как `image` **без растеризации** — сервер санитизирует (вырезает скрипты/обработчики) и хранит как `…/images/{hash}.svg`. Растр (`jpeg/png/webp/gif`) пережимается в `…/images/{hash}.webp`. SVG как `file` — только для скачивания, не для показа.

### Как загрузить локальный файл → Object Storage → CDN

MCP не читает файлы с диска агента (`C:\…`).  
Не вызывай `upload-media-from-fs` с Windows-path.  
`insert-media-block` файл не загружает. Сначала upload, потом insert.

**Запрещено**

- Писать свой upload (Python, Node).
- Использовать `upload-media-from-fs` / `recipe-upload-and-insert` с Windows-path или локальным image (`C:\…` — MCP сервер его не видит).
- Спрашивать пользователя «как залить».
- Класть файлы на VPS «под from-fs».
- Передавать `file_base64` в MCP.
- Заливать файл с кириллицей в пути или имени (сначала скопируй в ASCII-имя).
- Подменять в URL `originals` ↔ `processed` / `images` / выдуманные сегменты.
- Искать токен в переменных окружения шелла: там может лежать старый/отозванный токен. Токен для `curl` — тот же Bearer, что стоит в конфиге MCP-клиента в IDE (сервер `demonstrator-mcp` / `user-demonstrator-mcp`), или спроси у пользователя.

### Дерево решений: как загрузить файл

```
Файл — картинка для показа? (PNG/JPEG/WebP/GIF/SVG)
  ├─ Файл локально на твоём диске (C:\...)
  │   → upload-local-media.ps1 из текущего workspace — канон (сам вызывает curl локально).
  │   → ps1 рядом нет — Shell curl multipart (та же команда, ↓ см. «Картинка — curl / upload-local-media.ps1»).
  │   → НЕ вызывай recipe-upload-and-insert / upload-media-from-fs с file_path.
  │   → НЕ вызывай presign-media-upload (пресайн не для image).
  │
  ├─ Файл уже на публичном HTTPS
  │   → recipe-upload-and-insert с file_url + kind: image
  │
  └─ Файл на сервере MCP (не твоя машина)
      → upload-media-from-fs с file_path

Файл — аудио / видео / file / web_archive?
  ├─ Файл локально на твоём диске
  │   → presign-media-upload → Shell curl PUT → complete-media-upload
  │   → ИЛИ recipe-upload-and-insert если не нужно вручную PUT
  │
  ├─ Файл на публичном HTTPS
  │   → recipe-upload-and-insert с file_url
  │
  └─ Файл на сервере MCP
      → upload-media-from-fs с file_path
```

#### Выбор пути

| Что на диске агента | Как |
|---|---|
| Картинка для показа (`image`) | `upload-local-media.ps1` из текущего workspace (обёртка `curl`); нет рядом — Shell `curl` multipart (↓) |
| audio / video / file / web_archive | MCP `presign-media-upload` → Shell PUT → MCP `complete-media-upload` |
| Уже HTTPS URL | MCP `upload-media-from-fs` с `file_url` + `material_id`/`project_id` (+ верный `kind`) |
После upload вызови MCP `insert-media-block` или вставь `![…](full_url)` в content.  
Processing-медиа: `get-resource-status` до `ready` / `failed`.  
На каждом upload передай `material_id` или `project_id`.

**ASCII-only на диске.** Путь и имя файла для upload — только латиница/цифры (`C:\work\media\fig-01.png`).  
Кириллица в папках или имени файла ломает `curl` у агентов.  
Русское имя в медиатеке — после upload: `rename-resource` или `display_name` / `name` в `insert-media-block`.

#### Картинка — curl / upload-local-media.ps1

**Канон заливки картинки — multipart `curl` на `/api/resources`.** `upload-local-media.ps1` — обёртка для Windows из текущего workspace (`app/dev/website/agent/helpers/` или `tools/demonstrator-mcp/`). На Linux / macOS — тот же `curl`, не Python. Путь MCP `file_path` — путь на сервере MCP, не диск агента.

Канон локального image при наличии ps1 — вызвать **`upload-local-media.ps1`** (проверяет ASCII-путь/имя, возвращает JSON: `id`, `status`, `full_url`, `markdown_snippet`). Нет ps1 рядом — Shell `curl` multipart (та же команда ниже). Вызывать обёртку **вместо** ручного curl, когда она есть.

Если рядом ps1 нет (не тот пакет правил) — сделай тот же `curl` multipart вручную. Писать свой Python/Node upload нельзя.

Токен — тот же Bearer, что стоит в конфиге MCP-клиента в IDE (сервер `demonstrator-mcp` / `user-demonstrator-mcp`). **Не ищи токен в переменных окружения шелла** — там может лежать старый/отозванный токен после ротации. Если в конфиге IDE токена нет — спроси у пользователя. Prod API: `https://app.longread.agency`.

Если `POST /api/resources` отвечает `401` — токен не тот или ротирован: **стоп и спроси у пользователя актуальный токен**. Не перебирай варианты.

Через `upload-local-media.ps1`:

```powershell
.\upload-local-media.ps1 -FilePath C:\work\media\fig-01.png -Kind image -ProjectId 17 -Token "ТОКЕН_ИЗ_КОНФИГА_MCP"
```

Эквивалентный `curl`:

```powershell
curl.exe -sS -X POST "https://app.longread.agency/api/resources" `
  -H "Authorization: Bearer ТОКЕН_ИЗ_КОНФИГА_MCP" `
  -H "Accept: application/json" `
  -F "file=@C:\work\media\fig-01.png;filename=fig-01.png;type=image/png" `
  -F "project_id=17" `
  -F "original_name=fig-01.png"
```

1. Из JSON возьми `id` и `full_url` (или `url`) — путь `/images/…webp` (растр) или `/images/…svg` (SVG).
2. Вызови `insert-media-block` (`kind: image`) или вставь `![подпись](full_url)`.
3. Задай читаемое имя. Иначе вызови `rename-resource`.

Для image нет `presign-media-upload`.  
Нет MCP-tool, который читает локальный png — локальный файл видит только `upload-local-media.ps1` / `curl`.  
Не сканируй диск в поисках helper'а — он либо в этом workspace, либо делай curl.

#### audio / video / file / web_archive

1. MCP `presign-media-upload` (`kind`, `filename`, `size`, `material_id` или `project_id`).
2. Shell: `curl.exe -sS -X PUT "<upload_url>" -H "Content-Type: …" --data-binary "@C:\path\to\file"`.
3. MCP `complete-media-upload` (тот же `material_id` / `project_id`).
4. MCP `insert-media-block`.

`kind: file` — только для скачиваемых вложений. Не для логотипа/обложки/блока картинки.

#### Уже публичный HTTPS

MCP `upload-media-from-fs`: только `file_url` (https://…). Не `C:\…`. Для показа картинки — `kind: image`.

### Медиатека проекта

Медиатека = `resource_links` на `project_id` / `material_id`.

- Upload без `material_id`/`project_id` — MCP отклоняет вызов.
- При записи content через collab backend линкует твои ресурсы из markdown в материал/проект.
- Чужой CDN URL / demo-заполнитель без upload в медиатеку не попадает.

### Что не делать

- Не писать свои upload-скрипты сверх `upload-local-media.ps1` / `curl` выше.
- Не считать `insert-media-block` заменой upload.
- Не вызывать несуществующие base64-tools (`upload-media-resource` / `upload-resource`).
- Не класть медиа на диск VPS «под from-fs».
- Не подставлять Windows-path в `upload-media-from-fs`.
- Не загружать без `material_id` / `project_id`.
- Не заливать картинку для показа как `file` и не править CDN-путь вручную.
- Не вызывать MCP-инструменты для локального image (кроме `upload-local-media.ps1` / `curl`).

## ИИ-HTML (`:::ai_widget`)

| Инструмент | Действие |
|---|---|
| `create-widget` | Создать черновик ИИ-HTML. **Обязательно** `name` (или `caption` / `original_name`) — имя в медиатеке |
| `get-widget` | Получить ИИ-HTML; у `linear_dialog` — ещё `archetype` + `scenario` |
| `save-widget-draft` | Сохранить HTML в черновик (свободная вёрстка). **Не** для правок линейного диалога |
| `assemble-linear-dialog` | Собрать **или обновить** линейный диалог из полного `scenario` JSON (без Studio AI). Фича `ai_widget_templates` (тестеры) |
| `bake-widget` | Собрать ИИ-HTML в готовый архив |
| `fork-widget` | Форкнуть ИИ-HTML в другой материал |
| `insert-ai-widget-block` | Вставить `:::ai_widget` в материал |

### Имя в медиатеке

Ресурс без нормального `original_name` в UI выглядит как «Без названия» / бессмысленный дефолт. Это брак.

- `create-widget`: всегда передай `name` (коротко по смыслу: «Квиз онбординг», не uuid).
- Upload: осмысленный `filename`; если в библиотеке плохое имя — сразу `rename-resource`.
- `insert-media-block`: можно `display_name` / `name` — MCP переименует ресурс.

**Не готово:** виджет/файл в медиатеке без читаемого имени.

### Путь создания ИИ-HTML (только IDE + MCP)

```
локальный .html + маркер + отладка → вопрос «bake на CDN?» → «да» →
create-widget → save-widget-draft → bake-widget (ready) →
insert-ai-widget-block (fence = CDN entry_url, не id, не draft) →
poll archives/ до совпадения маркера → get-material
```

1. Запиши и отладь HTML в workspace (`widgets/<slug>.html`). Добавь уникальный маркер.
2. Спроси пользователя про bake/insert. Жди прямого «да». Не вызывай `bake-widget` на каждой правке.
3. `create-widget` → `save-widget-draft` (HTML с диска) → `bake-widget`.
4. **`insert-ai-widget-block`** — в теле `:::ai_widget` только **CDN `entry_url`** из `archives/` после статуса `ready`. Не bare id, не `draft_url`, не свежий `get-widget`.
5. Poll: `get-resource-status` / `archive-info` → HTML с `archives/` (`entry_url`, учти `?v=`). Маркер должен совпасть с локальным файлом. `get-widget` и draft — не критерий сдачи.
6. `get-material` — fence на месте. Путь к локальному файлу — пользователю.

В `create-widget` всегда передай **`name`** (имя в медиатеке). Без имени — не готово; при ошибке — `rename-resource`.

**Тема виджета** в `create-widget` / `save-widget-draft` — тот же **пресет** (палитра + стиль вместе), что и у материала. Не передавай одни `styleId` / `contentStyle`: палитра останется у дефолтного «Минимума» (`minimalism`), виджет выйдет серо-белым. Имена пресетов — `list-themes`; готовая палитра — в таблице ниже или через кастомный пресет проекта + `inherit-project-themes`. Подробнее — § «Темы» и `05-html-widget-rules.md`.

**Агент без IDE (Agentic):** локального `.html` нет. `save-widget-draft` получает HTML из поля/аргумента. После `bake-widget` (ready) и poll `archives/` — пришли человеку **CDN `entry_url` в чат** (не `draft_url`, не голый id). `insert-ai-widget-block` **не вызывай сам** — вставка в материал по отдельному «да». Детали — `05-html-widget-rules.md` («Финал: агент с MCP, но без IDE»).

### Шаблон: линейный диалог (`assemble-linear-dialog`)

Когда нужен **диалоговый плеер**, не пиши HTML с нуля — собери сценарий JSON и вызови `assemble-linear-dialog`. Shell канонический; агент заполняет **данные** (пул портретов, локации, реплики с парой слотов).

Предпочтительный HAPPY PATH: `recipe-linear-dialog` (создаёт виджет + собирает диалог + валидирует схему, ловит `portrait_url` и пропущенный `id`).

```
scenario.json на диске → recipe-linear-dialog (или create-widget → assemble-linear-dialog) →
вопрос «bake на CDN?» → «да» → bake-widget → insert-ai-widget-block (CDN URL)
```

#### Схема сценария (и частые ошибки)

Эталонный полный пример: `docs/templates/linear-dialog/scenario.example.json`. Основные поля:

| Поле | Правильно | Частая ошибка |
|---|---|---|
| `archetype` | `"linear_dialog"` **внутри `scenario`** | кладут в корень manifest'а, не в scenario |
| Персонаж — портрет | `imageUrl` + `portrait: "auto"` | `portrait_url` (несуществующий ключ) |
| Персонаж — ID | `id: "p1"` | нет поля `id` |
| Шаг — ID | `id: "s1"` обязательно | пропускают `id` |
| Шаг — локация | `locationId` обязательно | нет `locationId` → 422 |
| Шаг — портреты | `leftPortraitId`, `rightPortraitId` (два слота) | один слот или нет пары |
| Шаг — кто говорит | `speakerSide: "left"` или `"right"`, плюс `speaker` = id персонажа | нет speaker или неясно |
| Шаг — следующий | `next: "s2"` или `null` у последнего | все `null` — плеер остановится после первого |
| `locations[]` | **обязателен** (минимум одна, можно с пустым `imageUrl`) | не передают — 422 «опционально» в старых текстах ошибочно |
| `locations[].id` | `id: "loc1"` | нет id |

1. Сохрани `scenario` локально (`widgets/<slug>-scenario.json`). Начни с `docs/templates/linear-dialog/scenario.example.json` — не угадывай поля.
2. На сцене всегда **два слота картинок**; `characters[]` — N вариаций (ракурсы/эмоции). На каждой реплике задай пару портретов и кто говорит (`speakerSide`: `left`|`right`).
3. Демо-медиа CDN: `avatar-bot.webp`, `avatar-user.webp`, `slider-1.webp` на `cdn.longread.agency/demo/`.
4. `create-widget` с `name` → `assemble-linear-dialog` (`widget_id` + `scenario`). Не нужен `save-widget-draft`.
5. Bake/insert — только после прямого «да» (как у свободного HTML). Insert пишет **CDN `entry_url`**, не id.
6. При 403 `ai_widget_templates_forbidden` — шаблоны только у тестеров; не обходи через свободный HTML без согласования.

### Правка существующего линейного диалога

Тот же `assemble-linear-dialog` — **создаёт и обновляет**. Отдельного «refine» в MCP нет (Studio AI через MCP не вызываем).

```
get-widget → взять scenario → править JSON (локально или в ответе) →
assemble-linear-dialog (полный scenario) → при необходимости снова bake после «да»
```

1. `get-widget` (`widget_id`) → поля `archetype: linear_dialog` и `scenario`.
2. Если `scenario` пуст — вытащи из HTML `#vn-scenario` или восстанови из локального `widgets/<slug>-scenario.json`.
3. Внеси правки в **полный** scenario (реплики, портреты пула, `leftPortraitId`/`rightPortraitId`/`speakerSide`, локации, порядок `steps` / `next`). Не отправляй diff / SEARCH-REPLACE по shell.
4. Обнови локальный `widgets/<slug>-scenario.json`.
5. `assemble-linear-dialog` с тем же `widget_id` и **полным** `scenario` — пересоберёт draft.
6. **Запрещено** для диалога: `save-widget-draft` с ручным HTML плеера; правка CSS/JS shell; вызов `/api/ai/widgets/*/refine*`.
7. Повторный `bake-widget` / insert — только после прямого «да» пользователя (как при создании).

Чат-боты без MCP этот путь не используют. Они отдают HTML/markdown кодом и bake не делают.

**Не готово** (IDE): только bare id в fence; bake без «да»; bake без локального файла; bake без insert; сдача по draft при отстающем archive; fence с `draft_url`; ресурс в медиатеке без читаемого имени.

**Не готово** (агент без IDE): отдать `draft_url` или голый id вместо CDN; сдать по свежему `get-widget`, пока `archives/` без маркера; вставить `:::ai_widget` без запроса пользователя; ресурс в медиатеке без читаемого имени.

Серверный контракт — `05-html-widget-rules.md`.

## Корзина

| Инструмент | Действие |
|---|---|
| `list-project-trash` | Список удалённого |
| `restore-project-trash` | Восстановить (items: [{type, id}] или all: true) |

## Share-ссылки

Создание — да. Снятие и опасные обновления — да, но только после прямого «да» (см. README СТОП).

**Предпочти:** `recipe-share-link` (один вызов → канонический `url`). Примитивы `create-*-share-link` — запасной путь; при первой публикации **без** `refresh_only`.

| Инструмент | Действие | `destructiveHint` |
|---|---|---|
| `recipe-share-link` | HAPPY PATH: одна правильная ссылка (project/section/material). Сам решает first publish vs `refresh_only` | нет |
| `get-material-share-links` | Ссылки материала | нет |
| `create-material-share-link` | Создать (или взять существующую) ссылку материала | нет |
| `update-material-share-link` | Обновить `auto_update` у **preview** | **да** |
| `delete-material-share-link` | Снять ссылку материала (`preview` \| `preview_comments`) | **да** |
| `get-project-share-links` | Ссылки проекта | нет |
| `create-project-share-link` | Создать или обновить. **Первая публикация — без `refresh_only`** (сервер создаёт ссылку); `refresh_only: true` — только для уже опубликованной (иначе 404 «Проект не опубликован»). Предпочти `recipe-share-link` | нет |
| `delete-project-share-link` | Снять ссылку проекта + каскад (`link_type` обязателен) | **да** |
| `get-section-share-links` | Ссылки раздела | нет |
| `create-section-share-link` | Создать или обновить. **Первая публикация — без `refresh_only`**; `refresh_only: true` — только для уже опубликованной (иначе 404 «Раздел не опубликован») | нет |
| `delete-section-share-link` | Снять ссылку раздела + каскад (`link_type` обязателен) | **да** |
| `republish-comments-snapshot` | Обновить HTML-snapshot для **preview_comments**. **Не** пишет тело урока | **да** |

Ответ create/get содержит `url` — полный канонический URL.

Типы ссылок через MCP: только `preview` (просмотр) и `preview_comments` (с комментариями).

**`micro` («Микрообучение») через MCP нельзя** — ни создать, ни снять. Нужна ссылка «Микрообучение» → попроси человека в ЛК/UI. Не вызывай tool с `link_type: micro`.

**`recipe-share-link` — надёжный способ получить одну правильную ссылку** (project/section/material; `preview` \| `preview_comments`). Он сам решает: есть ссылка нужного типа → `refresh_only: true` (без ротации токенов); нет → первая публикация **без** `refresh_only`. Не давай агенту самому выбирать `refresh_only` для `create-*-share-link` — это ловушка, которая вела к 404 «не опубликован».

**Comment/share ≠ editor body.** `republish-comments-snapshot`, share-links с `preview_comments` и `get-material-preview-with-comments` — про публичный review/HTML. Текст урока в редакторе правят только `save-material-content` / `apply-material-point-patch`. В ответах материала нет внутренних publish-кэшей — только `content`.

## Чтение и экспорт

| Инструмент | Действие |
|---|---|
| `get-material` | id, title, content, theme, version, placement (whitelist) |
| `get-material-export-data` | Markdown + тема материала |
| `get-section-export-data` | Раздел + все материалы с контентом |
| `get-project-export-data` | Дерево проекта для HTML-ZIP |
| `get-material-preview-with-comments` | Read-only HTML snapshot+overlay; **не** источник для правки markdown |

## Предложения правок

Сейчас **вне MCP** (недоделано; вернём позже). Пиши тело сразу через collab (`save-material-content` / `apply-material-point-patch`).

## Запись контента

Источник правды при открытом редакторе — **Yjs collab**, не строка в БД. Все правки тела материала — через collab-мост.

| Инструмент | Действие |
|---|---|
| `apply-material-point-patch` | **Канон.** `replace_all` / `range_patch` / `find_replace_once` → `/api/collab/materials/:id/patch` |
| `save-material-content` | Полная замена тела тем же collab-мостом (`replace_all`) + опционально title/theme. По умолчанию `verify_readback: true` |
| `apply-material-theme` | Тема (palette / contentStyle / customOverlayCss) |

### Коридор при открытом редакторе

| Режим | Вкладка закрыта (`db_fallback`) | Вкладка открыта (живая комната) |
|---|---|---|
| `replace_all` / `save-material-content` | ок | ок → `yjs_room` (wipe всего файла, тост MCP). 409 возможна только при редкой гонке reconnect — повтори |
| `find_replace_once` | ок | ок → `yjs_room_point` (мелкий Y.Text op по точному совпадению **в живом** документе, не wipe) |
| `range_patch` | ок (одна-две мелкие правки) | **409 `point_patch_blocked_editor_open`** |

`find_replace_once` ищет якорь в live `Y.Text`, не в устаревшем GET из БД. Непересекающийся набор человека вне якоря сохраняется. `range_patch` по offsets с GET при открытой вкладке по-прежнему запрещён (splice). Массовая переписка урока — один `replace_all` + integrity.

### Режимы `apply-material-point-patch`

| Режим | Когда использовать |
|---|---|
| `replace_all` | Замена всего контента (last-writer-wins при открытом редакторе) |
| `range_patch` | Правка по смещению — только при **закрытой** вкладке |
| `find_replace_once` | Замена по точному тексту; **можно при открытом редакторе** |

Все write-операции используют `expected_version` (tool подставит сам, если не передан).

Ответ содержит `source`:
- `yjs_room` — открытый редактор, полная замена тела (wipe Y.Text);
- `yjs_room_point` — открытый редактор, точечный `find_replace_once` в live Y.Text;
- `db_fallback` — вкладка редактора не была открыта (нет живых WS).

`replace_all` / `save-material-content` при открытой вкладке: контент пишется в Yjs wipe+insert → приходит в Код как remote change (тост «обновлён с сервера (MCP)»). 409 (`editor_open_room_not_ready` / `collab_room_push_failed`) возможна только при редкой гонке reconnect — повтори запрос.

`range_patch` при открытой вкладке → **409 `point_patch_blocked_editor_open`**. Для точечной правки в открытом редакторе используй `find_replace_once`.

После записи смотри в ответе:

1. `verify.ok` — байтовое равенство отправленного и прочитанного (**не** «материал логически целый»).
2. `verify.integrity.ok` (и/или `integrity` в ответе collab patch) — структура: нет дублей `#`/`##`, нет голых `===` / orphan `:::/…`, нет незакрытых блоков, нет повторённого куска.
3. `source` и хвост `get-material` глазами при сомнении.

**Не сдавай**, если `verify.ok` true, а `integrity.ok` false. Надёжнее один финальный `replace_all` / `save-material-content` целым файлом, чем много мелких `find_replace_once`.

Не пишите тело через raw PATCH «в базу». Вторичных «workspace»/publish body-полей в ответах нет — править нечего, кроме `content` через collab.

## Темы

| Инструмент | Действие |
|---|---|
| `list-themes` | Список пресетов: minimalism, draft, journal, neon, airy, modern |
| `apply-material-theme` | Применить тему. Palette / `contentStyle` / `customOverlayCss`; также `bgImageResourceId` + `bgImageMode` (`pattern` \| `frame`) для фона-картинки из медиатеки |
| `inherit-project-themes` | Сбросить темы разделов/уроков проекта на наследование от темы проекта |

### Стиль ≠ пресет

В кабинете у темы две настройки: **«Стиль»** и **«Пресет»**.

| Что | Что несёт | Пример полей |
|---|---|---|
| **Пресет** | Тема целиком: палитра + стиль | accent, text, bg, surface, … + `styleId` / `contentStyle` |
| **Стиль** | Только оформление контента, **без** цветов | `{ styleId: "journal", contentStyle: "journal" }` |

Если у нового проекта, материала или виджета отправить тему из одних полей стиля, палитра пресета **не** подтянется: цвета останутся дефолтными чёрно-белыми от «Минимума» (`minimalism`). Ставь **пресет**, а не стиль.

Имена пресетов даёт `list-themes`. Палитры в ответе MCP нет — бери готовую тему с палитрой (кастомный пресет проекта + `inherit-project-themes`) или скажи человеку, чего не хватает, вместо частичной темы.

**Кастомный пресет проекта** (например «Журнал»): тема ставится на **проект**, уроки — через `inherit-project-themes`. Overlay пустой, пока человек не дал CSS. Копия `contentStyle: journal` на каждый урок при `styleId: default` задачу темы **не** закрывает — это не применение кастомного пресета проекта. Не выдумывай overlay и палитру, если у проекта уже есть пресет.

### Где лежит палитра пресетов

Цвета пресетов — в вебе редактора:

- `https://app.longread.agency/editor/js/design.js` — объект `THEME_PRESETS`
- `https://app.longread.agency/editor/js/theme-preset-overlays.js` — CSS оверлея стиля

Готовые значения (сверка с `design.js`):

| Пресет | accent | text | bg | surface | surfaceText | importantBg | positive | negative | border | styleId | contentStyle |
|---|---|---|---|---|---|---|---|---|---|---|---|
| journal | `#e91e63` | `#1f3249` | `#ffffff` | `#f8fafc` | `#1f3249` | `#faf5ff` | `#15803d` | `#b91c1c` | `#cad2dc` | `default` | `journal` |
| minimalism | `#3a3a3a` | `#222222` | `#ffffff` | `#f7f7f7` | `#222222` | `#f7f7f7` | `#20a020` | `#d04040` | `#e0e0e0` | `default` | `default` |
| draft | `#4a4a4a` | `#1f1f1f` | `#fafafa` | `#eeeeee` | `#1f1f1f` | `#eeeeee` | `#20a020` | `#d04040` | `#d0d0d0` | `sketch` | `sketch` |

У `journal` в теме ещё стоит `accentPale: "#cad2dc"` — платформа берёт бледный акцент из `border`, отдельно считать не нужно. Остальные пресеты (`neon`, `airy`, `modern`) — там же в `THEME_PRESETS`.

Признак, что пресет не применился, а лёг один стиль: палитра «Минимума» (`#3a3a3a`, `#f7f7f7`) — серый акцент вместо розового `#e91e63` и синего текста `#1f3249` у «Журнала».

`apply-material-theme` принимает произвольный theme object (merge через `MERGE_KEYS`). Фон-картинка:

- `bgImageResourceId` — id image-ресурса (или `null`, чтобы убрать);
- `bgImageMode` — `pattern` (плитка) или `frame` (фиксированный кадр); по умолчанию `pattern`.

Не подставляй картинку фона через `customOverlayCss` (`url(...)`) — используй поля темы. URL на shared/export резолвит платформа (`bg_image_url`).

Правила CSS для `customOverlayCss` — см. раздел «Правила CSS-генерации».
