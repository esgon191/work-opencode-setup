# work-opencode-setup

Конфигурация корпоративного агента для работы с набором скиллов.

# Первичная установка

```sh
git clone --recurse-submodules git@github.com:esgon191/work-opencode-setup.git
cd work-opencode-setup
```

Подмодуль `work-opencode-skills` должен содержать `pyproject.toml`: MCP-сервер
находится в ветке `mcp`, а не в старой ревизии ветки `master`.

# Настройка
## exports
### Путь к конфигу:
```sh
export OPENCODE_CONFIG_DIR=~/Desktop/work-opencode-setup/work-opencode-config # Без слеша в конце!
export OPENCODE_SETUP_DIR=~/Desktop/work-opencode-setup # Без слеша в конце!
```

### Подключения к БД

В `work-opencode-skills/connections.yaml` проверь хосты и порты. Затем создай
локальный файл с логином (без пароля) и установи зависимости:

```sh
cp "$OPENCODE_SETUP_DIR/work-opencode-skills/connections.local.yaml.example" \
   "$OPENCODE_SETUP_DIR/work-opencode-skills/connections.local.yaml"
# Укажи user в connections.local.yaml.

uv sync --project "$OPENCODE_SETUP_DIR/work-opencode-skills"
uv run --directory "$OPENCODE_SETUP_DIR/work-opencode-skills" \
  dwh-connections set-password gp_prod
uv run --directory "$OPENCODE_SETUP_DIR/work-opencode-skills" \
  dwh-connections check gp_prod
```

Пароль сохраняется в системном keyring и не записывается в файлы репозитория.

`uv run` ищет проект в текущем каталоге. Поэтому при запуске из корня
`work-opencode-setup` обязателен `--directory`: без него будет `Failed to spawn:
dwh-mcp`, даже если `.venv/bin/dwh-mcp` уже создан. Проверка самого MCP-сервера:

```sh
uv run --directory "$OPENCODE_SETUP_DIR/work-opencode-skills" dwh-mcp < /dev/null
```

Команда должна завершиться с кодом `0` без вывода. OpenCode использует созданный
`work-opencode-skills/.venv/bin/dwh-mcp` через абсолютный путь из конфигурации.

```
├── work-opencode-config/
└── work-opencode-skills/
    ├── connections.yaml                 контуры БД (хост, порт, база) — в git
    ├── connections.local.yaml           логин — локально, в .gitignore
```

### Креды для моделей:
```sh
export DEEPSEEK_API_KEY=
export GLM_API_KEY=  
```

### Креды для тулов: 
```sh
export JIRA_PERSONAL_TOKEN= 
export CONFLUENCE_PERSONAL_TOKEN= 
```
