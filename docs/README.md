# 📚 Документация VoidRP

Вики организации для разработчиков: как устроена платформа, как собрать любой репозиторий
и как сервисы общаются между собой. Для игроков — [гайды на сайте](https://void-rp.ru/server-guide).

| Раздел | О чём |
|---|---|
| [DEVELOPMENT.md](DEVELOPMENT.md) | Тулчейны, сборка и проверки каждого репозитория — теми же командами, что в CI |
| [INTEGRATION.md](INTEGRATION.md) | API бэкенда, секреты серверов, play-ticket, вход на плагинных серверах, WebGUI |
| [WEBGUI_ARCHITECTURE.md](WEBGUI_ARCHITECTURE.md) | Встроенный Chromium в клиенте: каналы, токены, страницы |
| [Схемы](../profile/diagrams) | PlantUML-исходники архитектуры и потока входа |
| [CONTRIBUTING](../CONTRIBUTING.md) · [SECURITY](../SECURITY.md) · [SUPPORT](../SUPPORT.md) | Как внести вклад, сообщить об уязвимости, получить помощь |

## Статус сборок

| Область | Репозиторий | CI | Тулчейн |
|---|---|---|---|
| Платформа | [minecraft-backend](https://github.com/VOIDRP-MINECRAFT/minecraft-backend) | [![CI](https://github.com/VOIDRP-MINECRAFT/minecraft-backend/actions/workflows/ci.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/minecraft-backend/actions/workflows/ci.yml) | Python 3.12 · FastAPI |
| Платформа | [voidrp-site](https://github.com/VOIDRP-MINECRAFT/voidrp-site) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-site/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-site/actions/workflows/build.yml) | Node 22 · Vue 3 |
| Лаунчер | [voidrp-launcher-vue](https://github.com/VOIDRP-MINECRAFT/voidrp-launcher-vue) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-launcher-vue/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-launcher-vue/actions/workflows/build.yml) | Node 22 · .NET 8 |
| Лаунчер | [voidrp-launcher-java](https://github.com/VOIDRP-MINECRAFT/voidrp-launcher-java) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-launcher-java/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-launcher-java/actions/workflows/build.yml) | Java 21 |
| Ванилла | [voidrp-ui](https://github.com/VOIDRP-MINECRAFT/voidrp-ui) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-ui/actions/workflows/ci.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-ui/actions/workflows/ci.yml) | Java 21 · Kotlin |
| Ванилла | [voidrp-auth-plugin](https://github.com/VOIDRP-MINECRAFT/voidrp-auth-plugin) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-auth-plugin/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-auth-plugin/actions/workflows/build.yml) | Java 25 |
| NeoForge | [voidrp-async-ai](https://github.com/VOIDRP-MINECRAFT/voidrp-async-ai) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-async-ai/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-async-ai/actions/workflows/build.yml) | Java 21 |
| NeoForge | [voidrp-anticheat](https://github.com/VOIDRP-MINECRAFT/voidrp-anticheat) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-anticheat/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-anticheat/actions/workflows/build.yml) | Java 21 |
| NeoForge | [voidrp-auth-bridge](https://github.com/VOIDRP-MINECRAFT/voidrp-auth-bridge) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-auth-bridge/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-auth-bridge/actions/workflows/build.yml) | Java 21 |
| NeoForge | [voidrp-webgui-neoforge](https://github.com/VOIDRP-MINECRAFT/voidrp-webgui-neoforge) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-webgui-neoforge/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-webgui-neoforge/actions/workflows/build.yml) | Java 21 |
| NeoForge | [voidrp-cpm-companion](https://github.com/VOIDRP-MINECRAFT/voidrp-cpm-companion) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-cpm-companion/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-cpm-companion/actions/workflows/build.yml) | Java 21 |
| NeoForge | [wg-region-guard](https://github.com/VOIDRP-MINECRAFT/wg-region-guard) | [![CI](https://github.com/VOIDRP-MINECRAFT/wg-region-guard/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/wg-region-guard/actions/workflows/build.yml) | Java 21 |
| NeoForge | [voidrp-client-fixes](https://github.com/VOIDRP-MINECRAFT/voidrp-client-fixes) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-client-fixes/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-client-fixes/actions/workflows/build.yml) | Java 21 |
| Paper | [voidrp-gamesync-plugin](https://github.com/VOIDRP-MINECRAFT/voidrp-gamesync-plugin) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-gamesync-plugin/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-gamesync-plugin/actions/workflows/build.yml) | Java 21 |
| Paper | [voidrp-battlepass](https://github.com/VOIDRP-MINECRAFT/voidrp-battlepass) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-battlepass/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-battlepass/actions/workflows/build.yml) | Java 21 |
| Paper | [voidrp-daily-quests](https://github.com/VOIDRP-MINECRAFT/voidrp-daily-quests) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-daily-quests/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-daily-quests/actions/workflows/build.yml) | Java 21 |
| Paper | [voidrp-wealth-tax](https://github.com/VOIDRP-MINECRAFT/voidrp-wealth-tax) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-wealth-tax/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-wealth-tax/actions/workflows/build.yml) | Java 21 |
| Paper | [voidrp-mod-sell](https://github.com/VOIDRP-MINECRAFT/voidrp-mod-sell) | [![CI](https://github.com/VOIDRP-MINECRAFT/voidrp-mod-sell/actions/workflows/build.yml/badge.svg)](https://github.com/VOIDRP-MINECRAFT/voidrp-mod-sell/actions/workflows/build.yml) | Java 21 |

<sub>В архиве: [voidrp-webgui](https://github.com/VOIDRP-MINECRAFT/voidrp-webgui) (Fabric-форк WebGUI).
Приватные репозитории в таблицу не входят.</sub>
