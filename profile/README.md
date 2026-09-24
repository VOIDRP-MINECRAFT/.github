# 🗡️ VoidRP Minecraft

> **Мультисерверная Minecraft-платформа: один аккаунт — разные миры. Модовый ролевой сервер с нациями и экономикой и ванильное выживание, на которое можно зайти с любого клиента.**

[![Server](https://img.shields.io/badge/Сайт-void--rp.ru-blueviolet?style=for-the-badge)](https://void-rp.ru)
[![Multi-server](https://img.shields.io/badge/Платформа-мультисервер-9b59b6?style=for-the-badge)](https://void-rp.ru/servers)
[![Mohist](https://img.shields.io/badge/Mohist-1.21.1-blue?style=for-the-badge)](https://mohistmc.com)
[![Paper](https://img.shields.io/badge/Paper-26.2-lightgrey?style=for-the-badge)](https://papermc.io)
[![Backend](https://img.shields.io/badge/API-FastAPI-009688?style=for-the-badge&logo=fastapi)](https://github.com/VOIDRP-MINECRAFT/minecraft-backend)
[![VoidRP UI](https://img.shields.io/badge/VoidRP_UI-Modrinth-00AF5C?style=for-the-badge&logo=modrinth&logoColor=white)](https://modrinth.com/plugin/voidrp-ui)

---

## 🌐 Платформа

Единый аккаунт VoidRP — **несколько игровых серверов**. Аккаунт-слой (логины, профили,
скины, донат, согласия) общий на всю платформу, а вся игровая механика (нации, экономика,
статистика, античит) **разделена по серверам** — у каждого свои данные.

| Сервер | Что это | Стек | Как зайти |
|---|---|---|---|
| **🏰 VoidRP** | Ролевой мир на модпаке FTB Evolution: нации и альянсы, динамическая экономика, рынок модовых предметов, боевой пропуск, квесты, странствующий торговец | Mohist 1.21.1 (NeoForge + Paper) | Лаунчер VoidRP |
| **🌱 Origins** | Ванильное выживание на плагинах: без модов и модпака, меню сервера и рынок игроков прямо на обычном клиенте | Paper 26.2 · Java 25 | Любой клиент 1.21.1 – 26.x или лаунчер |

---

## 🏗️ Стек технологий

```
┌──────────────────────────────────────────────────────────────────────┐
│                         VoidRP Architecture                          │
├──────────────┬───────────────────────────┬───────────────────────────┤
│   Launcher   │        Minecraft          │          Backend          │
│              │                           │                           │
│  Electron    │  VoidRP:  Mohist 1.21.1   │  FastAPI (Python)         │
│  Vue 3 · TS  │  Origins: Paper 26.2      │  PostgreSQL · Alembic     │
│  .NET 8      │  15+ своих модов          │  Redis · JWT              │
│  JavaFX 21   │  и плагинов               │  play-ticket auth         │
└──────────────┴───────────────────────────┴───────────────────────────┘
```

---

## 🗺️ Схема архитектуры

<div align="center">
<img src="diagrams/architecture.svg" alt="Архитектура VoidRP" width="920">
</div>

> Исходники схем — [`profile/diagrams/*.puml`](diagrams) (PlantUML). Стрелки подписаны протоколом/секретом, по которому идёт обмен.

---

## 🔬 Как это работает

### 1. Единый аккаунт, разделённые миры
Аккаунт-слой (`users` / `player_accounts`) — **глобальный**: один логин, профиль,
скин и донат на всю платформу. Вся игровая механика скоупится по `server_id →
game_servers` (30+ таблиц: нации, экономика, статистика, античит…). Набор разделов
для каждого сервера (моды, рынок, апгрейдер, торговец, нации, боевой пропуск)
включается флагами `game_servers.features`. Сервер определяется двумя путями:
- **плагины/моды** — по заголовку `X-Game-Auth-Secret` (у каждого сервера свой секрет);
- **сайт/лаунчер** — по `?server=<slug>` → `X-Server-Slug` → дефолтному серверу.

### 2. Лаунчер: манифест, синхронизация, помощь при крашах
Лаунчер (CoreHost на .NET 8) тянет с сервера **манифест пака** (генерится бэкендом
из `pack_root`, URL — в `game_servers.manifest_url`) и сверяет локальные файлы:
- скачивание **до 8 файлов параллельно** (HTTP/2), кэш SHA-256 не пересчитывает хеши зря;
- флаги файла: `alwaysOverwrite` (форс), `managed` (синк по хешу — обновление доходит,
  но неизменное не перекачивается); `config/` синкается по доставленным хешам, а клавиши
  и настройки графики игрока не перезаписываются;
- файлы каждого сервера изолированы в `servers/<slug>/`, место хранения игры выбирается в настройках;
- **краш-советник**: перед запуском проверяет память и диск, после краша сопоставляет лог
  с правилами от сервера и предлагает кнопки-исправления.

### 3. Вход без передачи пароля
- **Модовый сервер** — лаунчер по JWT запрашивает у бэкенда одноразовый `play-ticket`
  (короткий TTL), `auth-bridge` проверяет его при коннекте и получает профиль игрока,
  привязанный к `server_id`.
- **Плагинный сервер (Origins)** — игрок лаунчера входит сразу: билет едет меткой в адресе
  подключения. С обычного клиента `voidrp-auth-plugin` показывает **нативное окно
  Minecraft** «Вход на VoidRP» ещё до входа в мир, а пароль проверяет сайт — без AuthMe
  и без второй базы паролей. Там же можно зарегистрироваться.
- Билет выдаётся только после принятия актуальной оферты и согласия на обработку ПДн.

<div align="center">
<img src="diagrams/flow-auth-launch.svg" alt="Логин, синхронизация и вход на сервер" width="820">
</div>

### 4. Интерфейсы: WebGUI и VoidRP UI
- **Модовый клиент** — HTML/Vue-страницы поверх игры через встроенный Chromium (MCEF).
  Сервер шлёт клиенту `webgui:open_web` с URL и `webgui_token` (HMAC), клиент открывает
  `void-rp.ru/game-ui/*`, страница ходит в `/api/v1/game-ui/*`. Есть HUD с уведомлениями,
  гайдом новичка и дорожной картой.
- **Ванильный клиент** — [VoidRP UI](https://github.com/VOIDRP-MINECRAFT/voidrp-ui):
  панели со скруглениями, прозрачностью, шрифтом Inter, иконками предметов и курсором,
  который следует за мышью, — **без модов**. Страница пишется кодом, едет с сервера
  строкой текста в невидимом боссбаре и рисуется шейдерами ресурспака. Открытый
  Paper-плагин (MIT) на [Modrinth](https://modrinth.com/plugin/voidrp-ui) и
  [Hangar](https://hangar.papermc.io/mironoouv/VoidRP-UI).

### 5. Мод-прокси-шоп, экономика и торговец
На Mohist (NeoForge + Paper) EconomyShopGUI не умеет модовые предметы —
`gamesync-plugin` перехватывает транзакции, берёт цену из динамического
рыночного кэша (бэкенд-симуляция) и выдаёт предмет через `minecraft:give`.
Сделки конвертируются в XP боевого пропуска/квестов, комиссия идёт в казну нации.
По расписанию на спавне появляется **странствующий торговец** (NPC Citizens) с редкими
лотами, бюджетом выплат и открытием ассортимента по уровню боевого пропуска.

### 6. Античит и производительность
Серверный `anticheat` считает нарушения (Speed/Fly/Reach/KillAura/CPS) в VL,
шлёт репорты и снимки списка модов в бэкенд (админ выносит вердикты по модам).
`async-ai` — 60+ guard-миксинов против chunk-deadlock, зависаний главного потока и
падений от сломанных модов; `client-fixes` чинит клиентские краши модпака,
не требуя мода на сервере.

---

## 📦 Репозитории

### 🖥️ Инфраструктура

| Репозиторий | Описание | Стек |
|---|---|---|
| [minecraft-backend](https://github.com/VOIDRP-MINECRAFT/minecraft-backend) | REST API платформы: авторизация, мультисервер, нации, экономика, торговец, согласия, античит, админка | Python · FastAPI · SQLAlchemy · Alembic |
| [voidrp-site](https://github.com/VOIDRP-MINECRAFT/voidrp-site) | Сайт void-rp.ru: профили, нации, магазин, гайды по серверам, рейтинги, WebGUI-страницы, правовые документы | Vue 3 · Vite · Tailwind · daisyUI |

### 🚀 Лаунчер

| Репозиторий | Описание | Стек |
|---|---|---|
| [voidrp-launcher-vue](https://github.com/VOIDRP-MINECRAFT/voidrp-launcher-vue) | Десктопный лаунчер: выбор сервера, автообновление модпака, Java-рантайм, краш-советник | Electron · Vue 3 · TypeScript · .NET 8 |
| [voidrp-launcher-java](https://github.com/VOIDRP-MINECRAFT/voidrp-launcher-java) | Альтернативный лаунчер (standalone JAR) | JavaFX 21 · OkHttp · Gradle Shadow |

### 🌱 Плагины для ванильных серверов (Paper 26.2)

| Репозиторий | Описание |
|---|---|
| [voidrp-ui](https://github.com/VOIDRP-MINECRAFT/voidrp-ui) | ⭐ Настоящие интерфейсы на ванильном клиенте 1.21.6+: страницы в коде, курсор, темы, компоненты, API для своих плагинов. MIT |
| [voidrp-auth-plugin](https://github.com/VOIDRP-MINECRAFT/voidrp-auth-plugin) | Вход и регистрация через аккаунт сайта нативными окнами Minecraft; мгновенный вход из лаунчера |

### ⚔️ NeoForge моды (VoidRP, 1.21.1)

| Репозиторий | Описание |
|---|---|
| [voidrp-async-ai](https://github.com/VOIDRP-MINECRAFT/voidrp-async-ai) | Производительность: async AI, защита от deadlock/chunk-freeze и крашей модов (60+ миксинов) |
| [voidrp-anticheat](https://github.com/VOIDRP-MINECRAFT/voidrp-anticheat) | Серверный античит: Speed, Fly, Reach, KillAura, CPS + mod-snapshot |
| [voidrp-auth-bridge](https://github.com/VOIDRP-MINECRAFT/voidrp-auth-bridge) | Бридж авторизации (play-ticket), переподключения, система скинов с мгновенной сменой |
| [voidrp-cpm-companion](https://github.com/VOIDRP-MINECRAFT/voidrp-cpm-companion) | Косметика через CPM: гранты, экипировка, compositing скинов |
| [voidrp-webgui-neoforge](https://github.com/VOIDRP-MINECRAFT/voidrp-webgui-neoforge) | Встроенный Chromium WebGUI: HTML-страницы и HUD поверх игры, горячие клавиши |
| [voidrp-client-fixes](https://github.com/VOIDRP-MINECRAFT/voidrp-client-fixes) | Только клиент: crash-guard'ы и совместимость для модпака, на сервере не нужен |
| ~~[voidrp-webgui](https://github.com/VOIDRP-MINECRAFT/voidrp-webgui)~~ | _(архив — Fabric-форк, заменён voidrp-webgui-neoforge)_ |

### 🔌 Paper/Mohist плагины (VoidRP)

| Репозиторий | Описание |
|---|---|
| [voidrp-gamesync-plugin](https://github.com/VOIDRP-MINECRAFT/voidrp-gamesync-plugin) | Синхронизация статистики, прокси-шоп модовых предметов, динамические цены, торговец, гайд новичка, косметика |
| [voidrp-battlepass](https://github.com/VOIDRP-MINECRAFT/voidrp-battlepass) | Боевой пропуск: сезоны, 100 уровней + престиж, награды в Void Coins, x2 XP по выходным |
| [voidrp-daily-quests](https://github.com/VOIDRP-MINECRAFT/voidrp-daily-quests) | Ежедневные квесты с синхронизацией в WebGUI и интеграцией в экономику |
| [voidrp-wealth-tax](https://github.com/VOIDRP-MINECRAFT/voidrp-wealth-tax) | Прогрессивный налог на богатство для балансировки экономики |
| [voidrp-mod-sell](https://github.com/VOIDRP-MINECRAFT/voidrp-mod-sell) | Продажа предметов из модов через единый интерфейс |
| [wg-region-guard](https://github.com/VOIDRP-MINECRAFT/wg-region-guard) | Расширение WorldGuard: защита регионов и наций |

---

## 🌍 Особенности серверов

### 🏰 VoidRP — ролевой мир
- **Нации и альянсы** — территориальная политика, казна, дипломатия
- **Динамическая экономика** — рыночные цены, налоги, торговля модовыми предметами
- **Странствующий торговец** — NPC с редкими лотами по расписанию
- **Боевой пропуск + квесты** — сезонный прогресс, престиж и ежедневные задания
- **Гайд и дорожная карта** — подсказки новичкам прямо в HUD
- **Косметика** — CPM-модели с композитингом скина игрока

### 🌱 Origins — ванильное выживание
- **Без модов и лаунчера** — заходи с любого клиента от 1.21.1 до 26.x
- **Меню сервера на ванилле** — `/меню` рисуется через VoidRP UI на клиенте 1.21.6+: гайд, новости, топ игроков, настройки, RU/EN
- **Рынок игроков** — тот же, что на сайте: `/shop` прямо в игре
- **Вход окном Minecraft** — пароль от аккаунта VoidRP, регистрация прямо в игре
- **Классика** — дома, телепорты к друзьям, PvP вне спавна, вещи выпадают при смерти; приватов нет, гриф запрещён правилами

### 🔧 Общее для платформы
- **Мультисервер** — один аккаунт, изолированные игровые данные по серверам
- **Античит** — серверная верификация движения + snapshot загруженных модов
- **60+ performance-миксинов** — нулевые chunk-deadlock'и, async AI, guard'ы для проблемных модов
- **Play-ticket auth** — безопасная авторизация лаунчер → сервер → API
- **Согласия и документы** — оферта и политика ПДн по 152-ФЗ, публичность профиля по выбору игрока

---

## 📊 Технические достижения

```
✅ Мультисервер            — общий аккаунт, per-server игровые данные (30+ таблиц)
✅ UI на ванильном клиенте  — страницы кодом через боссбар + шейдеры, без модов
✅ Вход окнами Minecraft    — серверные диалоги 1.21.6+ поверх аккаунта сайта
✅ Chunk-deadlock guards    — 20+ паттернов заблокировано
✅ Async watchdog           — автодиагностика и автофикс через Claude AI
✅ Mod proxy shop           — NeoForge предметы в Paper-магазине без рестарта
✅ Live skin compositing    — серверная генерация скинов (CPM)
✅ Crash advisor            — правила крашей от сервера и исправления в один клик
✅ Play-ticket auth         — безопасная авторизация лаунчер → сервер → API
```

---

<div align="center">

**[🌐 Сайт](https://void-rp.ru)** · **[🖥️ Серверы](https://void-rp.ru/servers)** · **[💬 Discord](https://discord.gg/voidrp)** · **[🗺️ Карта](https://void-rp.ru/map)**

*VoidRP — твой мир, твои правила*

</div>
