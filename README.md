---
title: Mini App preview — single-file HTML
purpose: тыкабельный прототип Mini App (4 экрана, mock-данные) для Жени и команды
status: preview
created: 2026-05-02
owner: designer
tags: [ui, miniapp, preview, html, mock]
---

# Mini App preview

Single-file HTML прототип Mini App SellerAssistantBot. Открывается двойным кликом из Проводника или через localtunnel/ngrok как Telegram Web App.

## Что внутри

`index.html` — один файл с inline CSS и vanilla JS. Без сборки, без зависимостей кроме Telegram WebApp SDK с CDN (`https://telegram.org/js/telegram-web-app.js`).

### 4 экрана

1. **Главная** (`main` / `escalations-list`) — приветствие по времени дня, аватар, 3 KPI (открытых эскалаций / закрыто за день / среднее время), 4 цветные action-карточки (Эскалации / Чаты / Контрагенты / Метрики), список «В РАБОТЕ» с прогресс-барами и приоритет-полосками, секция «НЕДАВНО». MainButton снизу: «Ответить на 3 затыка».
2. **Деталь эскалации** (`escalation-detail`) — шапка контрагента (тап → contragent-detail), summary с приоритетной полоской, контекст чата (5 bubbles, триггер подсвечен), композер (voice / text / recorded), кнопка «Снять без ответа», MainButton «Отправить ответ».
3. **Список чатов** (`chats-list`) — фильтр-чипы (Все / Auto / Silent / Paused), карточки чатов с avatar-кружком, режим-бейдж (тап → bottom-sheet с переключением режима), переход на карточку контрагента.
4. **Карточка контрагента** (`contragent-detail`) — hero (аватар / специализация / локация / since), метрики 7 дней (4 карточки), открытые вопросы (чек-лист), открытые эскалации, все чаты, стиль коммуникации (буллеты), договорённости (timeline), прецеденты, контакты (phone/email/tg).

### Mock данные

Все данные inline в JS-объекте `MOCK_DATA` в начале `<script>`. Реалистичные сущности:

- Селлер: Сергей.
- Контрагенты: ФФ Alice (Алиса-Логистик, Электросталь), ФФ Bob, Виктор (ЭлектроСталь-ФФ), ФФ Дима, ФФ Сергей П., ABC-Логистика (новый, без карточки).
- Чаты: Test_FF_Direct, Test_FF_Group, ЭлектроСталь-ФФ, Балашиха-Логистика, Подольск-Партнёры, ABC-Логистика.
- Поставки: 4421, 4419, 4310. SKU: 89234561, 89234581, 87651234.
- 3 mock эскалации: недовоз 4421 (priority 1), скидка 8% (priority 2), расхождение Alice/Bob (priority 3).

## Как открыть локально

### Вариант 1 — двойной клик
Открыть `index.html` в браузере. Telegram-фичи (BackButton, MainButton-native) недоступны — эмулируется через CSS/JS fallback'ы. Layout центрируется в 420px колонке на десктопе.

### Вариант 2 — через localtunnel (для Telegram)
```
npx localtunnel --port 8000
# в другом окне:
python -m http.server 8000
```
Полученный URL засунуть в BotFather → Edit Bot → Bot Settings → Menu Button → URL.

### Вариант 3 — через ngrok
```
ngrok http 8000
```

## Дизайн-язык

Адаптировано из MarketplaceAI TMA (см. `MarketplaceAI/03_AGENTS/applied/tma/src/components/`):

- Тёмная тема: `#0F1620` фон, `#1A2332` surface, `#232E3C` surface-2.
- KPI-блоки 3-в-ряд: 22px bold цифры, 11px серая подопись, цветной delta-чип (зелёный +12% / красный -1.4пп).
- Action-cards 2×2: linear-gradient (orange / purple / teal / pink), эмоджи-иконка 38px в скруглённом квадрате слева, белый bold заголовок + opacity-подопись, chevron справа.
- Task-cards: вертикальная цветная полоска слева (приоритет), прогресс-бар, status-pill (Выполняется / В очереди / Готово / Ошибка), timestamp справа.
- Скругления 16px, тени мягкие, шрифт system-ui (San Francisco / Roboto).

## Routing

Простая state-машина (`state.currentScreen` + `state.history` + `state.params`). Без отдельных HTML-файлов. Back-навигация через `back()` (синхронизирована с Telegram BackButton API если запущено в TG).

## Telegram WebApp интеграция

- `tg.ready()` + `tg.expand()` при старте.
- Темные/светлые цвета из `tg.themeParams` перекрывают дефолтные через CSS-переменные.
- `tg.BackButton` show/hide на не-главных экранах + обработка `backButtonClicked`.
- `tg.showConfirm` для confirm-диалогов («Снять без ответа?»).
- В обычном браузере fallback'ы: BackButton рендерится как обычная `← Назад` кнопка слева сверху, confirm через `window.confirm`, MainButton как fixed-bottom CSS-блок.

## Известные ограничения / компромиссы

- **Mock везде**: ни одна кнопка не дёргает реальный n8n webhook. «Отправить ответ» / «Применить режим» / «Снять без ответа» показывают toast и возвращают назад. Никаких backend-запросов.
- **Voice recording — симуляция**: тап на voice-кнопку через 800ms «записал» голос с захардкоженным STT-текстом (`оформи возврат на 8 шт sku 89234561`). MediaRecorder API не подключён.
- **Нет skeleton/loading state**: все данные рендерятся мгновенно из MOCK_DATA. Loading/error состояния из round-2 спеки описаны, но в preview не реализованы (нет фактической загрузки).
- **Action card «Контрагенты» и «Метрики недели»**: тап выводит toast «в разработке (mock)» — отдельных экранов нет. Только Эскалации (главный) и Чаты ведут куда-то.
- **Group-чат** (`Test_FF_Group`) при тапе показывает toast вместо action-sheet выбора контрагента.
- **Inline-кнопка `[+карточка]`** для «нового чата без карточки» — через toast, без отдельного flow.
- **Pull-to-refresh** не реализован (нет источника обновления).
- **Поиск в `chats-list`** не реализован (по дизайну на пилоте 20 чатов — без поиска ок).
- **Empty/Error states**: для filter «Paused (0)» работает empty-state. Глобальный error-state не воспроизводится.
- **Footer «Открыть в Obsidian»** не показывается в preview (специфично для Жени с vault).

## Размер

`index.html` — около 41 KB inline (без CDN-скрипта Telegram SDK).

## Файлы

| Файл | Назначение |
|---|---|
| `index.html` | главный файл preview, single-file |
| `README.md` | это описание |

## История

- 2026-05-02: первая версия preview (designer agent, round 3)
