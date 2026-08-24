[🇷🇺 Русский](README.md) · [🇬🇧 English](README_EN.md)

# 🛠 GMod Server Helper

> Сколько весит твоя коллекция? И что кинуть в `workshop.lua`? Хелпер отвечает на оба вопроса.

**🔴 Live-демо:** https://duckflit.github.io/gmod_helper/

Набор бесплатных браузерных инструментов для админов серверов Garry's Mod.
Без бэкенда, API-ключей и регистрации: вся страница — один `index.html`,
который можно открыть локально или выложить на GitHub Pages.

<!-- Когда добавишь скриншот, раскомментируй и положи файл в docs/screen.png
![Скриншот](docs/screen.png)
-->

## 🧰 Инструменты

| Инструмент | Статус | Что делает |
|---|---|---|
| **Collection Helper** | ✅ работает | Вес каждого аддона коллекции, суммарный вес, самый тяжёлый аддон, сортировка и экспорт в CSV |
| **Workshop.lua генератор** | ✅ работает | Генерирует готовый `workshop.lua` со строками `resource.AddWorkshop(...)` из коллекции — с кнопкой скачивания |
| Проверка конфликтов | 🔜 скоро | Поиск конфликтов моделей/материалов/энтити |
| server.cfg генератор | 🔜 скоро | Готовый конфиг с комментариями |
| Оценка нагрузки | 🔜 скоро | Прикидка нагрузки коллекции на сервер |

Интерфейс — **русский и английский** (переключатель в сайдбаре, выбор запоминается).

## 🚀 Как пользоваться

1. Открой https://duckflit.github.io/gmod_helper/ (или локальный `index.html`).
2. Вставь ссылку на коллекцию или её ID (например `2812330501`).
3. Нажми **«Анализировать»** — получишь веса и статистику,
   или **«Сгенерировать»** — получишь файл `workshop.lua`.

Пример сгенерированного файла:

```lua
-- workshop.lua · GMod Server Helper
-- Collection: https://steamcommunity.com/sharedfiles/filedetails/?id=2812330501
-- Addons: 299 · 2025-08-24

resource.AddWorkshop("3690602885") -- Пособие по атмосфернагагой физике
resource.AddWorkshop("1604765873") -- !*$%? ERRORS
resource.AddWorkshop("246756300") -- 3D Stream Radio
```

Файл кладётся в `garrysmod/lua/autorun/server/workshop.lua` — сервер подтянет аддоны при старте.

## ⚙️ Как это работает

- Страница коллекции читается через публичные CORS-прокси и разбирается прямо в браузере (`DOMParser`).
- Размеры берутся из публичного Steam Web API (`ISteamRemoteStorage/GetPublishedFileDetails`) —
  **POST-запросом**: Steam требует именно POST, и хелпер это учитывает.
- Если один прокси лежит — автоматически пробуются другие; в «Журнале отладки» видно каждый шаг.
- Скрытые, забаненные и удалённые аддоны не имеют публичного размера — они помечаются отдельно и не входят в сумму.

## 🖥 Запуск и деплой

```bash
# Локально — просто открыть файл
index.html

# На GitHub Pages
git add . && git commit -m "gmod server helper" && git push
# Settings → Pages → Branch: main → Save
```

## 🔐 Свой CORS-прокси (опционально)

Публичные прокси иногда лежат. Для 100% стабильности разверни свой на
Cloudflare Workers (бесплатно, 15 строк):

<details>
<summary>Код воркера</summary>

```js
export default {
  async fetch(req){
    const url = new URL(req.url);
    if (req.method === 'OPTIONS'){
      return new Response(null, { status: 204, headers: {
        'Access-Control-Allow-Origin': '*', 'Access-Control-Allow-Headers': '*',
        'Access-Control-Allow-Methods': 'GET,POST,OPTIONS' } });
    }
    if (url.pathname !== '/proxy')
      return new Response('proxy alive', { headers: { 'Access-Control-Allow-Origin': '*' } });
    const target = url.searchParams.get('url');
    if (!target) return new Response('missing url', { status: 400 });
    const res = await fetch(target, {
      method: req.method,
      body: (req.method === 'POST' || req.method === 'PUT') ? req.body : undefined,
      headers: { 'Content-Type': req.headers.get('Content-Type') || 'application/x-www-form-urlencoded' },
    });
    return new Response(res.body, { status: res.status, headers: {
      'Access-Control-Allow-Origin': '*',
      'Content-Type': res.headers.get('Content-Type') || 'application/json' } });
  },
};
```

</details>

Затем вставь URL воркера в `index.html`: `const MY_PROXY = 'https://имя.workers.dev';`

## 🗂 Структура

```
gmod_helper/
├── index.html      # всё приложение: стили, разметка, логика
├── README.md       # ты здесь (RU)
└── README_EN.md    # английская версия
```

## 🗺 Roadmap

- [ ] Проверка конфликтов аддонов
- [ ] Генератор server.cfg
- [ ] Оценка нагрузки коллекции
- [ ] Сохранение коллекций в localStorage

## ⚠️ Дисклеймер

Проект не аффилирован с Valve Corporation и Steam. Все данные берутся из публичных
страниц Steam и Steam Web API.
