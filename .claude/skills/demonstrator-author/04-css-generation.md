---
id: 04-css-generation
updated: 2026-08-11 03:40 GMT+3
---

# Правила CSS тем (`customOverlayCss`)

CSS для `apply-material-theme` → `customOverlayCss`. Это **ограничения**, не гайд по дизайну.

## Фон страницы (shared / export)

В shared колонка текста — `body` (max-width 720px), а **холст страницы** — `html` на весь viewport. Класс `.longread-theme-custom` вешается и на `html`, и на `body`; у колонки `background: transparent !important`.

Канон shared красит холст так:

```css
html:has(body.shared-view) {
  background-color: var(--longread-bg);
  background-image: var(--longread-bg-image, none);
  background-size: var(--longread-bg-image-size, 960px auto);
}
```

Специфичность `html:has(body.shared-view)` выше, чем у `.longread-theme-custom`. Один только `background: …gradient…` на классе **не** перекрывает канон: на `html` остаётся solid `--longread-bg` и `background-image: none`.

| Задача | Как |
|---|---|
| Плоский фон | `--longread-bg` — **только solid** `#hex` (нижний слой и для `color-mix`). |
| Картинка из медиатеки | Поля темы `bgImageResourceId` + `bgImageMode` (`pattern` \| `frame`). Не пиши `url(...)` руками в overlay. |
| Градиент на весь viewport | `--longread-bg-image` + `--longread-bg-image-size: 100% 100%` на `.longread-theme-custom` (см. ниже). |
| Не делать | Селекторы `html` / `body`; `gradient(...)` в `--longread-bg`; считать задачу решённой после одного `background: …gradient…` на классе без переменных и без проверки shared. |

### Градиент на всё окно (shared / export)

1. `theme.bg` / `--longread-bg` — **только solid**.
2. Визуальный градиент на весь viewport задавай через переменные, которые читает канон холста:

```css
.longread-theme-custom {
  --longread-bg-image:
    radial-gradient(
      ellipse 80% 50% at 70% 0%,
      color-mix(in srgb, var(--longread-accent) 40%, transparent),
      transparent 70%
    ),
    linear-gradient(
      180deg,
      color-mix(in srgb, var(--longread-surface) 85%, var(--longread-accent)) 0%,
      var(--longread-bg) 100%
    );
  /* иначе дефолт 960px auto — плитка, не «на всё окно» */
  --longread-bg-image-size: 100% 100%;
}
```

3. Без `--longread-bg-image` холст shared останется плоским, даже если на `.longread-theme-custom` есть `background: …gradient…`.
4. Не ставь селекторы `html` / `body`. Не клади `gradient(...)` в `--longread-bg`.
5. `--longread-bg-image` — тот же хук, что у картинки из медиатеки (`bgImageResourceId`). Свой градиент в переменной перебивает page-image — авторская ответственность. С `bgImageMode: frame` full-bleed градиент через эту var может вести себя иначе; проверяй shared без frame либо не мешай с frame.
6. Перед сдачей — **shared-view** (hard refresh): у `html` computed `background-image` содержит `gradient`, `background-size` — на весь холст (`100% 100%` или эквивалент), а не пятна только на узкой колонке.

**Антипаттерн:** `background: radial-gradient…` на `.longread-theme-custom` без `--longread-bg-image` / `--longread-bg-image-size` и без проверки shared.

### Штатный фон-картинка (тема)

- `bgImageResourceId` — id ресурса `kind=image` в медиатеке (или `null` / пусто = без картинки).
- `bgImageMode`:
  - `pattern` — плитка (wallpaper), `background-size` от холста 960px; на узком экране crop, не scale-down.
  - `frame` — один кадр fixed-слоем `#longread-page-bg` (не `background-attachment: fixed`).
- URL резолвит платформа (`bg_image_url` / `theme.bgImageUrl` на shared). В `customOverlayCss` **не** дублируй картинку через `url(...)`.
- Solid `--longread-bg` остаётся под картинкой (в т.ч. под прозрачным PNG).
- Свой `--longread-bg-image` (градиент) в overlay перебивает page image — авторская ответственность.

## Цвета — только переменные

Цвета темы материала задаются через `--longread-*`. В `customOverlayCss` для **любого** цвета (`color`, `background`, `background-color`, `border-color`, `outline-color`, `box-shadow` с цветом, `text-shadow` с цветом, `fill`, `stroke`) пиши **только** `var(--longread-…)`.

| Переменная | Назначение |
|---|---|
| `--longread-bg` | Базовый фон страницы (solid) |
| `--longread-text` | Текст |
| `--longread-surface` | Фон карточек / поверхностей |
| `--longread-surface-text` | Текст на surface |
| `--longread-headings` | Заголовки |
| `--longread-accent` | Акцент |
| `--longread-accent-pale` | Бледный акцент |
| `--longread-border` | Границы |
| `--longread-positive` | «Хорошо» |
| `--longread-negative` | «Плохо» |
| `--longread-important-bg` | Фон important |

Отступы темы (не цвета, но тоже из темы): `--longread-content-padding`, `--longread-block-padding`, `--longread-block-margin`.

### Запрещено (цвета)

- Голые `#hex`, `rgb()`, `rgba()`, `hsl()`, именованные цвета (`white`, `red`, …) в свойствах цвета.
- Свои `--my-*` / хардкод «под палитру клиента» вместо `--longread-*`.
- Менять значения переменных в `:root` / `html` / `body` из overlay (палитра — полями theme API, не CSS).

```css
/* Верно */
.longread-theme-custom h1 { color: var(--longread-headings); }
.longread-theme-custom .block.btn a {
  background: var(--longread-accent);
  color: var(--longread-surface);
}

/* Запрещено */
.longread-theme-custom h1 { color: #1f3249; }
.longread-theme-custom .card { background: white; }
```

Исключение: прозрачность через `color-mix` / `oklch` **от** `var(--longread-*)` — ок. Чистый hex — нет. Иной цвет без переменной — только после прямого «да» пользователя.

Self-lint перед `apply-material-theme`: в CSS нет `#` / `rgb(` / `rgba(` / `hsl(` для цветов (кроме редкого согласованного исключения).

## Селекторный контекст

- Каждое правило — с префикса `.longread-theme-custom`.
- Запрещены селекторы: `body`, `html`, `:root`.

## Не ломай вёрстку

Без прямого «да» пользователя не трогай: `display`, `position`, `flex`, `grid`, `width`, `height`, `overflow`.

Не стилизуй без «да»: `.test-options`, radio/checkbox внутри теста, их `::before` / `::after` / `::marker`.

Ограничения:

- `.block.important` — `padding-left` ≥ `3em`; не ломай `::before` (иконка).
- `.flip-front` / `.flip-back` — не убирай `padding-bottom`.

Безопасный коридор по умолчанию: `color`, `background`, `border*`, `border-radius`, `box-shadow`, `text-shadow`, `font-*`, `letter-spacing`, `padding`/`margin` (осторожно), `transform`.

## Классы блоков (шпаргалка селекторов)

Стилизуй только нужное. Цвета — снова только `var(--longread-*)`.

| Блок | Селекторы |
|---|---|
| Заголовки | `h1`…`h6` |
| Важное | `.block.important`, модификаторы `.idea` / `.problem` / `.solution` / `.info` / `.rule` |
| Аккордеон | `.accordion`, `.accordion-item`, `.accordion-head`, `.accordion-body` |
| Вкладки | `.tabs-block`, `.tabs-nav`, `.tabs-panel` |
| Карточки | `.flip-front`, `.flip-back` |
| Тест | `.test-block`, `.test-question`, `.test-feedback` |
| Шаги | `.steps-item`, `.steps-num`, `.steps-title`, `.steps-body` |
| Чеклист | `.block.checklist`, `.checklist-item` |
| Сравнение | `.compare-col-good`, `.compare-col-bad` |
| Кнопка | `.block.btn a` |
| Прочее | `.block.img img`, `.block.audio`, `table`/`th`/`td`, `.block.embed`, `.comments-block` |

Вложенные `blockquote` (три уровня) — не ломай структуру; цвета уровней тоже через `--longread-*`.
