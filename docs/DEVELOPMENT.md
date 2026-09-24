# Сборка и разработка

Команды ниже взяты из CI каждого репозитория (`.github/workflows/*.yml`): если они проходят
локально, пройдут и в CI.

## Тулчейны

| Что | Версия |
|---|---|
| JDK для NeoForge-модов и Paper-плагинов 1.21.1 | **21** (Temurin) |
| JDK для плагинов под Paper 26.2 (`voidrp-auth-plugin`) | **25** — Gradle подтянет toolchain сам |
| VoidRP UI | собирается на **21**, компилируется против Paper API 26.2 |
| Python (бэкенд) | **3.12** |
| Node.js (сайт, лаунчер) | **22** |
| .NET (ядро лаунчера) | **8.0** |

## Команды по репозиториям

### NeoForge-моды и Paper-плагины 1.21.1

`voidrp-async-ai`, `voidrp-anticheat`, `voidrp-auth-bridge`, `voidrp-webgui-neoforge`,
`voidrp-cpm-companion`, `wg-region-guard`, `voidrp-gamesync-plugin`, `voidrp-wealth-tax`,
`voidrp-mod-sell`, `voidrp-launcher-java`:

```bash
./gradlew build --stacktrace --no-daemon
```

> [!NOTE]
> Если в `gradle.properties` прописан локальный прокси разработчика, CI вырезает его строкой
> `sed -i "/[Pp]roxy/d" gradle.properties`. Не коммитьте свой прокси.

`voidrp-battlepass` и `voidrp-daily-quests` зависят от соседних плагинов: в CI рядом клонируются
`voidrp-gamesync-plugin` (и `voidrp-mod-sell` для квестов) и собираются первыми.

### VoidRP UI

```bash
./gradlew test          # тесты
./gradlew shadowJar     # build/libs/*-all.jar
./gradlew preview       # отрисовать страницы в PNG без запуска игры
```

Релиз: тег `v*` → GitHub Actions собирает jar и прикладывает к релизу.

### voidrp-auth-plugin (Paper 26.2)

```bash
./gradlew shadowJar     # build/libs/voidrp-auth-1.0.0-all.jar
```

### voidrp-client-fixes

Гард для Jade компилируется против `libs/Jade-1.21.1-NeoForge-*.jar` (`compileOnly`, в репозиторий
не коммитится). Положите этот файл с [Modrinth](https://modrinth.com/mod/jade) в `libs/` — CI
скачивает ровно его через Modrinth API — и соберите `./gradlew build`.

### minecraft-backend

```bash
pip install -e ".[dev]"
python -m compileall apps
pytest -q
```

Миграции — Alembic (`alembic/versions`).

### voidrp-site

```bash
yarn install --frozen-lockfile
yarn build
yarn dev --host         # dev-сервер; /api и /media проксируются на прод-API
```

### voidrp-launcher-vue

```bash
npm ci
npm run build:core:dev:linux   # CoreHost (.NET 8)
npm run build:electron
npm run build:renderer
```

## Релизы

Во всех Gradle-репозиториях тег `v*` запускает тот же CI, и собранный jar прикладывается к
GitHub-релизу с автоматическими release notes:

```bash
git tag v1.2.3 && git push origin v1.2.3
```

## Обновления зависимостей

В каждом репозитории настроен Dependabot (`.github/dependabot.yml`): раз в неделю, по понедельникам,
одним сгруппированным PR на экосистему — Gradle, npm, pip, NuGet и GitHub Actions. PR проходит тот же
CI. Мажорные версии и сама платформа (Paper, NeoForge) в эти PR не попадают — их обновляют вручную
вместе с версией сервера.

## Секреты

В репозиториях — только плейсхолдеры. Реальные значения живут на серверах:

| Секрет | Где задаётся | Кто использует |
|---|---|---|
| `X-Game-Auth-Secret` | админка → Серверы (у каждого сервера свой) | моды и плагины при запросах к бэкенду |
| WebGUI HMAC-секрет | `config/webgui/server.json` | `gamesync-plugin` и `webgui-neoforge` для `webgui_token` |
| JWT-ключи, БД, Redis | окружение бэкенда | `minecraft-backend` |
