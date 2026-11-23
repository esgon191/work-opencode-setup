# work-opencode-setup

Основной репозиторий конфигурации корпоративного OpenCode для работы с DWH,
Jira и Confluence. Он объединяет совместимые ревизии конфигурации и DWH
MCP-сервера через Git-подмодули.

## Состав

- `work-opencode-config/` — конфигурация OpenCode, провайдеры моделей, MCP и
  разрешения; исходная ветка подмодуля — `agents`.
- `work-opencode-skills/` — локальный MCP-сервер `dwh-mcp` для Greenplum и
  ClickHouse; исходная ветка подмодуля — `mcp`.

## Установка

Полный порядок первичной установки, настройки и проверки описан в
[INSTALL.md](INSTALL.md).

Необходимые компоненты: Git, OpenCode, uv и Python 3.10 или новее.

Кратко для новой установки в `~/Desktop`:

```sh
cd ~/Desktop
git clone --recurse-submodules git@github.com:esgon191/work-opencode-setup.git
cd work-opencode-setup
```

После клонирования следуй [INSTALL.md](INSTALL.md): задай переменные окружения,
настрой логин DWH, установи зависимости через `uv sync`, сохрани пароли в keyring
и проверь запуск `dwh-mcp` перед стартом OpenCode.

## Обновление

Основной репозиторий фиксирует совместимые ревизии подмодулей. Обновляй их через
него:

```sh
cd "$OPENCODE_SETUP_DIR"
git pull
git submodule update --init --recursive
uv sync --project "$OPENCODE_SETUP_DIR/work-opencode-skills"
```

После обновления перезапусти OpenCode.

## Дополнительная документация

- [Конфигурация OpenCode](work-opencode-config/README.md).
- [DWH MCP: инструменты и разработка](work-opencode-skills/README.md).

Пароли и токены не хранятся в репозитории. Пароли DWH сохраняются в системном
keyring, а токены передаются через переменные окружения.
