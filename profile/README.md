# 🗡️ VoidRP Minecraft

> **Кастомный Minecraft-сервер с ролевым игровым миром — экономика, нации, косметика и хардкорный геймплей**

[![Server](https://img.shields.io/badge/Сервер-void--rp.ru-blueviolet?style=for-the-badge)](https://void-rp.ru)
[![Version](https://img.shields.io/badge/Minecraft-1.21.1-green?style=for-the-badge&logo=minecraft)](https://minecraft.net)
[![NeoForge](https://img.shields.io/badge/NeoForge-21.1.x-orange?style=for-the-badge)](https://neoforged.net)
[![Paper](https://img.shields.io/badge/Mohist-Paper%20%2B%20Forge-blue?style=for-the-badge)](https://mohistmc.com)

---

## 🏗️ Стек технологий

```
┌─────────────────────────────────────────────────────────────────┐
│                        VoidRP Architecture                      │
├──────────────┬──────────────────────┬───────────────────────────┤
│   Launcher   │     Minecraft        │         Backend           │
│              │                      │                           │
│  Electron    │  Mohist 1.21.1       │  FastAPI (Python)         │
│  Vue 3       │  NeoForge + Paper    │  PostgreSQL               │
│  .NET 8      │  16+ кастомных       │  Redis                    │
│  JavaFX 21   │  модов и плагинов    │  JWT Auth                 │
└──────────────┴──────────────────────┴───────────────────────────┘
```

---

## 📦 Репозитории

### 🖥️ Инфраструктура

| Репозиторий | Описание | Стек |
|---|---|---|
| [minecraft-backend](https://github.com/VOIDRP-MINECRAFT/minecraft-backend) | REST API сервера: авторизация, нации, экономика, античит | Python · FastAPI · SQLAlchemy · Alembic |
| [voidrp-site](https://github.com/VOIDRP-MINECRAFT/voidrp-site) | Сайт void-rp.ru: профили, нации, магазин, карта | Vue 3 · Vite · Tailwind · daisyUI |

### 🚀 Лаунчер

| Репозиторий | Описание | Стек |
|---|---|---|
| [voidrp-launcher-vue](https://github.com/VOIDRP-MINECRAFT/voidrp-launcher-vue) | Десктопный лаунчер с автообновлением модпака | Electron · Vue 3 · TypeScript · .NET 8 |
| [voidrp-launcher-java](https://github.com/VOIDRP-MINECRAFT/voidrp-launcher-java) | Альтернативный лаунчер (standalone JAR) | JavaFX 21 · OkHttp · Gradle Shadow |

### ⚔️ NeoForge моды (серверные)

| Репозиторий | Описание |
|---|---|
| [voidrp-async-ai](https://github.com/VOIDRP-MINECRAFT/voidrp-async-ai) | Производительность: async AI, защита от deadlock/chunk-freeze (60+ миксинов) |
| [voidrp-anticheat](https://github.com/VOIDRP-MINECRAFT/voidrp-anticheat) | Серверный античит: Speed, Fly, Reach, KillAura, CPS + mod-snapshot |
| [voidrp-cpm-companion](https://github.com/VOIDRP-MINECRAFT/voidrp-cpm-companion) | Косметика через CPM: гранты, экипировка, compositing скинов |
| [voidrp-auth-bridge](https://github.com/VOIDRP-MINECRAFT/voidrp-auth-bridge) | Бридж авторизации: play-ticket flow прямо в игре |
| [voidrp-webgui](https://github.com/VOIDRP-MINECRAFT/voidrp-webgui) | Встроенный веб-интерфейс (мульти-версия) |
| [voidrp-webgui-neoforge](https://github.com/VOIDRP-MINECRAFT/voidrp-webgui-neoforge) | Веб-GUI для NeoForge 1.21.1 |

### 🔌 Paper/Mohist плагины

| Репозиторий | Описание |
|---|---|
| [voidrp-gamesync-plugin](https://github.com/VOIDRP-MINECRAFT/voidrp-gamesync-plugin) | Синхронизация статистики, прокси-шоп для модовых предметов, динамические цены |
| [voidrp-battlepass](https://github.com/VOIDRP-MINECRAFT/voidrp-battlepass) | Боевой пропуск: сезонные задания, XP, награды |
| [voidrp-daily-quests](https://github.com/VOIDRP-MINECRAFT/voidrp-daily-quests) | Ежедневные квесты с интеграцией в экономику |
| [voidrp-wealth-tax](https://github.com/VOIDRP-MINECRAFT/voidrp-wealth-tax) | Прогрессивный налог на богатство для балансировки экономики |
| [voidrp-mod-sell](https://github.com/VOIDRP-MINECRAFT/voidrp-mod-sell) | Продажа предметов из модов через единый интерфейс |
| [wg-region-guard](https://github.com/VOIDRP-MINECRAFT/wg-region-guard) | Расширение WorldGuard: защита регионов и нации |

---

## 🌍 Особенности сервера

- **Нации и альянсы** — территориальная политика, казна, дипломатия
- **Динамическая экономика** — рыночные цены, налоги, торговля модовыми предметами
- **Боевой пропуск** — сезонный прогресс с ежедневными квестами
- **Косметика** — CPM-модели с композитингом скина игрока
- **Античит** — серверная верификация движения + snapshot загруженных модов
- **60+ performance-миксинов** — нулевые chunk-deadlock'и, async AI, guard'ы для всех проблемных модов

---

## 📊 Технические достижения

```
✅ Chunk-deadlock guards    — 20+ паттернов заблокировано
✅ Async watchdog           — автодиагностика и автофикс через Claude AI
✅ Mod proxy shop           — NeoForge предметы в Paper-магазине без рестарта
✅ Live skin compositing    — серверная генерация CPM-моделей со скином игрока
✅ Play-ticket auth         — безопасная авторизация лаунчер → сервер → API
```

---

<div align="center">

**[🌐 Сайт](https://void-rp.ru)** · **[💬 Discord](https://discord.gg/voidrp)** · **[🗺️ Карта](https://void-rp.ru/map)**

*VoidRP — твой мир, твои правила*

</div>
