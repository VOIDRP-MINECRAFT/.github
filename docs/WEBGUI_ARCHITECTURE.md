# WebGUI Architecture

Архитектура встроенного Chromium-браузера в Minecraft-клиенте VoidRP.

---

## Обзор

VoidRP использует [WebGUI](https://github.com/VOIDRP-MINECRAFT/voidrp-webgui) + [MCEF](https://github.com/Keksuccino/MCEF) для отображения HTML/CSS/Vue-страниц прямо внутри Minecraft. Это позволяет заменить Bukkit inventory GUI на полноценные веб-интерфейсы.

```
  Minecraft Server (Mohist 1.21.1 — NeoForge + Paper)
  ├── voidrp-webgui-neoforge     регистрирует каналы с .optional()
  └── voidrp-gamesync-plugin     транспортирует пакеты через sendPluginMessage()
        │
        │  Bukkit plugin channels
        │  webgui:open_web      VarInt(1) + VarInt(mode) + MCString(url_with_token)
        │  webgui:set_main_menu MCString(url)
        ▼
  Minecraft Client
  ├── voidrp-webgui              NeoForge jar (через Connector) — рендер браузера
  └── mcef-keksuccino-2.2.0      Chromium CEF — Connector грузит для нативок
        │
        │  HTTPS запросы к API
        ▼
  void-rp.ru/game-ui/*           Vue 3 SPA страницы
        │  ?webgui_token=<HMAC-SHA256>
        ▼
  minecraft-backend (FastAPI)    /api/v1/game-ui/* роутеры
```

---

## Компоненты

### 1. NeoForge мод — [voidrp-webgui-neoforge](https://github.com/VOIDRP-MINECRAFT/voidrp-webgui-neoforge)

Серверный мод. Регистрирует сетевые каналы и обрабатывает события страниц.

**Ключевые детали:**
- Все каналы регистрируются с `.optional()` — handshake проходит даже если одна сторона не поддерживает канал
- `displayTest="IGNORE_ALL_VERSION"` — исключает дисконнект при несовпадении версий мода
- Выходной jar: `build/libs/webgui-1.3.0+mc1.21.1.jar` (75 КБ)

**Каналы:**
```
webgui:open_web       S2C — открыть URL в GUI или HUD режиме
webgui:set_main_menu  S2C — задать URL для клавиши F6
webgui:emit           S2C — отправить событие на страницу
webgui:page_event     C2S — получить событие от страницы
```

### 2. Paper-плагин мост — [voidrp-gamesync-plugin](https://github.com/VOIDRP-MINECRAFT/voidrp-gamesync-plugin)

`WebGuiBridgeService` — **единственный транспорт** для отправки URL клиентам. Bukkit не может диспатчить NeoForge-команды, поэтому пакеты уходят через `player.sendPluginMessage()` raw bytes.

```java
// Открыть fullscreen страницу
webGuiBridge.openGui(player, "https://void-rp.ru/game-ui#market");

// HUD-оверлей поверх игры
webGuiBridge.openHud(player, "https://void-rp.ru/game-ui/hud");

// URL для клавиши F6
webGuiBridge.sendMainMenuUrl(player, "https://void-rp.ru/game-ui/menu");
```

`signUrl()` добавляет `?webgui_token=<signed>` перед отправкой. Токен живёт 7200 секунд (2 часа).

### 3. Клиентский мод — [voidrp-webgui](https://github.com/VOIDRP-MINECRAFT/voidrp-webgui)

Fabric-форк WebGUI v1.3.0 с VoidRP-кастомизациями. На сервере и в лаунчер-паке используется нативный NeoForge jar из `voidrp-webgui-neoforge`.

**VoidRP расширения (page→game JS-каналы):**
```js
// Выполнить команду от имени игрока
postToGame({ channel: "run_command", command: "/pm pickup" })

// Открыть другой раздел без round-trip к серверу
postToGame({ channel: "open_gui", url: "https://void-rp.ru/game-ui#treasury" })

// Обновить HUD
postToGame({ channel: "open_hud", url: "https://void-rp.ru/game-ui/hud" })
```

**MCEF зеркало:** CEF нативки (~119 МБ) раздаются с `https://void-rp.ru/launcher/mcef` через `config/mcef.properties` в лаунчер-паке.

### 4. Backend — [minecraft-backend](https://github.com/VOIDRP-MINECRAFT/minecraft-backend)

Четвёртый слой авторизации — `webgui_token`. Роутер `game_ui_market.py` с `webgui_auth` dependency.

**Эндпоинты `/api/v1/game-ui/market/`:**
- `GET order-book/{item_key}` — книга ордеров
- `GET my-orders` — мои ордера
- `GET items` — список товаров  
- `GET trades` — история сделок
- `POST pending-action` — создать действие для плагина (buy, cancel, pickup)
- `GET pickup-ready` — незабранные доставки

**Таблица `player_market_web_actions`** — очередь действий от браузера к плагину (TTL 3-5 минут, поллинг раз в секунду через `WebActionPollService`).

### 5. Фронтенд — [voidrp-site](https://github.com/VOIDRP-MINECRAFT/voidrp-site)

`/game-ui/*` маршруты с `hidePublicShell: true` (нет навбара/футера). Авторизация через `?webgui_token=`.

**Composables (`src/composables/useWebGui.js`):**
```js
const token = useWebGuiToken()           // строка токена из URL
const client = useWebGuiClient()         // реактивный { playerUuid, username, ... }
const inGame = isInMod()                 // true если открыто внутри Minecraft
await postToGame({ channel: "..." })     // управление игрой через CEF bridge
```

---

## Токен (auth flow)

```
1. Игрок входит → WebGuiPlayerJoinListener (60-tick delay)
         ↓
2. WebGuiBridgeService.sendMainMenuUrl()
   signUrl("https://void-rp.ru/game-ui/menu")
         ↓
3. payload = base64url("1|nickname|expiresAt")
   token = payload + "." + base64url(HMAC-SHA256(payload, secret))
         ↓
4. Пакет webgui:set_main_menu → клиент
         ↓
5. Игрок жмёт F6 → браузер открывает URL с токеном
         ↓
6. JavaScript читает ?webgui_token= → все API запросы идут с токеном
         ↓
7. FastAPI: webgui_auth.py верифицирует HMAC → lookup PlayerAccount
```

Секрет хранится в двух местах: `config/webgui/server.json` (NeoForge мод + Paper-плагин) и `WEBGUI_TOKEN_SECRET_BASE64` в `.env` бэкенда.

---

## Pending Web Actions (Vault-транзакции из браузера)

Браузер не имеет доступа к Vault. Схема для операций с деньгами:

```
Браузер: POST /game-ui/market/pending-action { action_type: "buy", ... }
         ↓
backend создаёт PlayerMarketWebAction (status=pending, expires_at=+5min)
         ↓
WebActionPollService (раз в секунду): GET /game-sync/market-web-actions
         ↓
Плагин: Vault.withdraw() → backend.ackAction(id) → создать ордер
```

**Типы action_type:**
- `buy` — списать деньги через Vault, создать buy order
- `cancel_sell` — вернуть предмет, удержать комиссию 0.5%
- `cancel_buy` — вернуть деньги (нетто за вычетом комиссии)
- `pickup` — выдать pending доставки (ack before deliver)

Для операций с аналогами в командах (`/pm cancel`) лучше использовать `run_command` JS-канал — мгновенно, без задержки поллинга.

---

## Статус реализации

| Компонент | Статус |
|---|---|
| NeoForge мод (каналы, `.optional()`) | ✅ Готово |
| `displayTest="IGNORE_ALL_VERSION"` | ✅ Готово |
| WebGuiBridgeService (Paper-плагин) | ✅ Готово |
| WebGuiPlayerJoinListener | ✅ Готово |
| WebActionPollService | ✅ Готово |
| webgui_auth.py (HMAC-SHA256) | ✅ Готово |
| `/game-ui/market` (backend + frontend) | ✅ Готово |
| `/game-ui/menu` (frontend) | ✅ Готово |
| MCEF зеркало | ✅ Готово |
| VoidRP page→game JS-каналы | ✅ Готово |
| Vue composables (useWebGui.js) | ✅ Готово |
| `/game-ui/hud` | 🔄 Запланировано |
| `/game-ui/nation-market` | 🔄 Запланировано |
| `/game-ui/treasury` | 🔄 Запланировано |
| `/game-ui/battlepass` | 🔄 Запланировано |
| `/game-ui/quests` | 🔄 Запланировано |
| `/game-ui/alliance` | 🔄 Запланировано |

---

## Добавление нового раздела — чеклист

1. **Фронтенд:** новый Vue view + маршрут `/game-ui#<section>` в router
2. **Главное меню:** кнопка в `GameUiMenuView.vue` с `postToGame({channel:"open_gui", url:"...#<section>"})`
3. **Backend:** новый FastAPI router с `get_webgui_player` dependency
4. **Плагин (если нужна команда):** одна строка в `WebGuiBridgeService` + диспатч
5. **Плагин (если нужен Vault):** новый `action_type` в таблице + обработчик в `WebActionPollService`

---

*Подробный план: [WEBGUI_INTEGRATION_PLAN.md](https://github.com/VOIDRP-MINECRAFT/voidrp-gamesync-plugin/blob/main/WEBGUI_INTEGRATION_PLAN.md)*
