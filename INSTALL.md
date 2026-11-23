# Установка work-opencode-setup

Эта инструкция описывает первичную установку конфигурации OpenCode, локального
MCP-сервера `dwh-mcp` и подключений к Greenplum и ClickHouse.

Выполняй шаги последовательно. Команды рассчитаны на установку в
`~/Desktop/work-opencode-setup`; при выборе другого каталога измени пути в шагах 2 и 3.

## 1. Проверь необходимые программы и доступы

Нужны:

- Git и SSH-доступ к репозиториям на GitHub;
- OpenCode;
- [uv](https://docs.astral.sh/uv/);
- Python 3.10 или новее;
- логин и пароли от контуров DWH;
- токены используемых моделей и, при необходимости, Jira и Confluence.

Проверка команд:

```sh
git --version
opencode --version
uv --version
```

## 2. Склонируй репозиторий вместе с подмодулями

```sh
cd ~/Desktop
git clone --recurse-submodules git@github.com:esgon191/work-opencode-setup.git
cd work-opencode-setup
```

Если репозиторий уже склонирован без подмодулей:

```sh
cd ~/Desktop/work-opencode-setup
git submodule update --init --recursive
```

Проверь, что установлена ревизия `work-opencode-skills` с MCP-сервером:

```sh
test -f work-opencode-skills/pyproject.toml
```

Отсутствие вывода и код `0` означают успех. Если файла нет, обнови основной
репозиторий и повтори `git submodule update --init --recursive`.

## 3. Задай пути для OpenCode

Для текущей сессии, находясь в корне `work-opencode-setup`:

```sh
export OPENCODE_SETUP_DIR="$PWD"
export OPENCODE_CONFIG_DIR="$OPENCODE_SETUP_DIR/work-opencode-config"
```

Обе переменные должны содержать абсолютные пути **без `/` на конце**. Они нужны
до запуска OpenCode: конфигурация использует их для поиска `dwh-mcp`.

Для постоянной настройки добавь в `~/.zshrc` абсолютные пути, например:

```sh
export OPENCODE_SETUP_DIR="$HOME/Desktop/work-opencode-setup"
export OPENCODE_CONFIG_DIR="$OPENCODE_SETUP_DIR/work-opencode-config"
```

После изменения `~/.zshrc` открой новый терминал или выполни `source ~/.zshrc`.

## 4. Настрой контуры и логин DWH

Проверь хосты, порты и имена баз в `work-opencode-skills/connections.yaml`.
Меняй их только если данные устарели.

Создай локальный файл с логином:

```sh
cp -n "$OPENCODE_SETUP_DIR/work-opencode-skills/connections.local.yaml.example" \
  "$OPENCODE_SETUP_DIR/work-opencode-skills/connections.local.yaml"
```

В `connections.local.yaml` замени `your.login` на свой логин. Если для одного из
контуров нужен другой логин, используй секцию `databases` из примера в файле.

**Пароль в `connections.local.yaml` не записывай.** Он хранится отдельно в
системном keyring.

## 5. Установи Python-зависимости

```sh
uv sync --project "$OPENCODE_SETUP_DIR/work-opencode-skills"
```

Команда создаёт `work-opencode-skills/.venv` и исполняемые файлы `dwh-mcp` и
`dwh-connections`.

## 6. Сохрани пароли и проверь подключения

Greenplum:

```sh
uv run --directory "$OPENCODE_SETUP_DIR/work-opencode-skills" \
  dwh-connections set-password gp_prod
uv run --directory "$OPENCODE_SETUP_DIR/work-opencode-skills" \
  dwh-connections check gp_prod
```

ClickHouse, если он нужен:

```sh
uv run --directory "$OPENCODE_SETUP_DIR/work-opencode-skills" \
  dwh-connections set-password ch_prod
uv run --directory "$OPENCODE_SETUP_DIR/work-opencode-skills" \
  dwh-connections check ch_prod
```

Каждая команда `set-password` интерактивно запрашивает пароль и сохраняет его в
системном keyring. Список контуров и состояние логинов и паролей можно посмотреть
командой:

```sh
uv run --directory "$OPENCODE_SETUP_DIR/work-opencode-skills" dwh-connections
```

## 7. Задай токены моделей и Atlassian

Перед запуском OpenCode экспортируй токен выбранной модели:

```sh
export DEEPSEEK_API_KEY="<токен DeepSeek>"
export GLM_API_KEY="<токен GLM>"
```

Достаточно задать токен используемой модели. Для MCP Atlassian дополнительно:

```sh
export JIRA_PERSONAL_TOKEN="<токен Jira>"
export CONFLUENCE_PERSONAL_TOKEN="<токен Confluence>"
```

Эти переменные также нужно сохранить в окружении, из которого запускается
OpenCode. Не добавляй реальные токены и пароли в файлы репозитория.

## 8. Проверь запуск dwh-mcp

```sh
uv run --directory "$OPENCODE_SETUP_DIR/work-opencode-skills" dwh-mcp < /dev/null
```

Команда должна завершиться с кодом `0` без сообщений об ошибке. Это только
проверка запуска: в обычном режиме `dwh-mcp` работает через stdio и запускается
самим OpenCode.

OpenCode использует абсолютный путь:

```text
$OPENCODE_SETUP_DIR/work-opencode-skills/.venv/bin/dwh-mcp
```

## 9. Запусти OpenCode и проверь инструменты

```sh
opencode
```

В OpenCode попроси показать доступные инструменты DWH. Должны быть доступны
инструменты с префиксами `dwh_gp_` и `dwh_ch_`, включая `dwh_gp_connections` и
`dwh_ch_connections`.

Запросы `dwh_gp_query_run` и `dwh_ch_query_run`, возвращающие данные, требуют
подтверждения. Не запрашивай через них персональные данные.

## Обновление

Обновление выполняется из основного репозитория, чтобы получить зафиксированные
совместимые ревизии подмодулей:

```sh
cd "$OPENCODE_SETUP_DIR"
git pull
git submodule update --init --recursive
uv sync --project "$OPENCODE_SETUP_DIR/work-opencode-skills"
```

После обновления перезапусти OpenCode.

## Устранение ошибок

### `Failed to spawn: dwh-mcp` или `Failed to spawn: dwh-connections`

Не запускай `uv run dwh-mcp` из корня `work-opencode-setup` без указания проекта.
Используй:

```sh
uv run --directory "$OPENCODE_SETUP_DIR/work-opencode-skills" dwh-mcp < /dev/null
```

Также проверь наличие `work-opencode-skills/pyproject.toml`. Если его нет,
выполни обновление по инструкции выше.

### `MCP error -32000: Connection Closed`

Проверь пути и исполняемый файл:

```sh
printf '%s\n' "$OPENCODE_SETUP_DIR" "$OPENCODE_CONFIG_DIR"
test -x "$OPENCODE_SETUP_DIR/work-opencode-skills/.venv/bin/dwh-mcp"
```

Если исполняемого файла нет, повтори шаг 5. Путь должен быть абсолютным: `~` и
`$HOME` внутри конфигурации OpenCode не раскрываются.

Логи OpenCode находятся в `~/.local/share/opencode/log/`.

### Логин не задан или пароль не найден

Проверь `connections.local.yaml` и повтори соответствующую команду
`dwh-connections set-password` из шага 6.
