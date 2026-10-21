# work-opencode-setup

Набор конфигов, инструментов и установщиков для корпоративного опенкода с фокусом на задачи продуктовой аналитики.

## Структура

Репозиторий собирает два сабмодуля:

```
work-opencode-setup/
├── work-opencode-config/   конфиг opencode (opencode.jsonc): модели, разрешения, MCP-серверы
└── work-opencode-skills/   локальный MCP-сервер dwh (Greenplum и ClickHouse)
    ├── connections.yaml            контуры БД (хост, порт, база) — в git
    ├── connections.local.yaml      логин — локально, в .gitignore
    ├── pyproject.toml
    └── src/dwh_mcp/
        ├── server.py               MCP-сервер (stdio), описания инструментов
        ├── connections.py          подключение к БД + CLI dwh-connections
        ├── ddl.py / tables.py      gp_table_ddl, gp_schema_tables
        ├── query.py                gp_query_test, gp_query_run
        └── ch_*.py                 аналогичные инструменты для ClickHouse
```

Клонирование:

```sh
git clone --recursive git@github.com:esgon191/work-opencode-setup.git
# репозиторий уже склонирован без сабмодулей:
git submodule update --init --recursive
```

## Переменные окружения

| Переменная | Назначение |
|---|---|
| `OPENCODE_CONFIG` | путь к файлу конфига opencode (см. ниже) |
| `DEEPSEEK_API_KEY` | креды модели deepseek |
| `GLM_API_KEY` | креды модели glm |
| `JIRA_PERSONAL_TOKEN` | MCP-сервер Atlassian (Jira) |
| `CONFLUENCE_PERSONAL_TOKEN` | MCP-сервер Atlassian (Confluence) |
| `OPENCODE_SETUP_DIR` | Абсолютный путь к данному репозиторию на устройстве |

Блок для `~/.zshrc` / прописать в терминале, где будет запущен opencode:

```sh
# путь к конфигу opencode, если склонировать репозиторий на рабочий стол    
export OPENCODE_CONFIG=~/Desktop/work-opencode-setup/work-opencode-config/opencode.jsonc

# креды для моделей
export DEEPSEEK_API_KEY=
# если GLM не доступен - удалить провайдера ecom/glm из конфига
export GLM_API_KEY=

# креды для тулов
export JIRA_PERSONAL_TOKEN=
export CONFLUENCE_PERSONAL_TOKEN=
```

> `OPENCODE_CONFIG` указывает на **файл** конфига. Не путать с `OPENCODE_CONFIG_DIR`:
> та переменная задаёт каталог для `agents/`, `commands/`, `plugins/` (структура как у
> `.opencode/`) и сам `opencode.jsonc` из него не подхватывает.

## Настройка dwh-mcp

Нужен [uv](https://docs.astral.sh/uv/). `uv run` ищет проект в текущей папке —
запускай из корня репозитория skills или указывай `--directory`, иначе будет
`failed to spawn: dwh-connections`.

```bash
cd ~/Desktop/work-opencode-setup/work-opencode-skills
cp connections.local.yaml.example connections.local.yaml   # указать user
uv run dwh-connections set-password gp_prod                # пароль в keyring
uv run dwh-connections check gp_prod                       # проверка соединения
uv run dwh-connections set-password ch_prod                # пароль ClickHouse
uv run dwh-connections check ch_prod
```

Из любой папки: `uv run --directory <путь к репозиторию>/work-opencode-setup/work-opencode-skills dwh-connections check gp_prod`.

## Подключение к OpenCode

В `work-opencode-config/opencode.jsonc` сервер dwh описан так:

```jsonc
    "mcp": {                                                                               
    "atlassian": {                                                                         
      "type": "local",                                                                     
      "command": ["uvx", "mcp-atlassian==0.21.1"],                                         
      "environment": {                                                                     
        "JIRA_URL": "https://project.samokat.ru",                                          
        "JIRA_PERSONAL_TOKEN": "{env:JIRA_PERSONAL_TOKEN}",                                
                                                                                           
        "CONFLUENCE_URL": "https://space.samokat.ru",                                      
        "CONFLUENCE_PERSONAL_TOKEN": "{env:CONFLUENCE_PERSONAL_TOKEN}",                    
        "READ_ONLY_MODE": "true"                                                           
        }                                                                                  
      },                                                                                   
    "dwh": {                                                                               
      "type": "local",                                                                     
      // "command": ["/Users/hq-fvfkjlu11wfv/Desktop/mcp/work-opencode-skills/.venv/bin/dwh-mcp"],
      "command": ["{env:OPENCODE_SETUP_DIR}/work-opencode-setup/work-opencode-skills/.venv/bin/dwh-mcp"],
      "enabled": true                                                                      
    }                                                                                      
  }                                                                                        

```

Путь к `dwh-mcp` — только **абсолютный**: opencode запускает команду без shell,
`~` и `$HOME` не раскрываются, и сервер падает с `MCP error -32000: Connection Closed`.
Получить путь: `echo "$PWD/.venv/bin/dwh-mcp"` в корне репозитория skills (venv
создаётся после первого `uv sync` / `uv run`).

Вариант через переменную окружения — в `command` можно подставить `{env:OPENCODE_SETUP_DIR}`
(с `export OPENCODE_SETUP_DIR=~/Desktop/work-opencode-setup` в `~/.zshrc`):
`["{env:OPENCODE_SETUP_DIR}/work-opencode-skills/.venv/bin/dwh-mcp"]`. Перед
использованием проверь, что подстановка отрабатывает: `opencode debug config`.

Вариант через uv (подтягивает зависимости сам после `git pull`) — тоже с
абсолютными путями, `uv` может отсутствовать в `PATH` процесса opencode:
`["/Users/<you>/.local/bin/uv", "run", "--directory", "/Users/<you>/Desktop/work-opencode-setup/work-opencode-skills", "dwh-mcp"]`.

Проверка, что инструменты видны: перезапустить `opencode` и спросить, какие есть
инструменты dwh. Ожидаемо: `dwh_gp_connections`, `dwh_gp_table_ddl`,
`dwh_gp_schema_tables`, `dwh_gp_query_test`, `dwh_gp_query_run` и `dwh_ch_*` —
те же пять для ClickHouse.

Если сервер не поднимается — смотри логи opencode в `~/.local/share/opencode/log/`.

## Инструменты dwh

| Инструмент | Что делает |
|---|---|
| `gp_connections` / `ch_connections` | список контуров, логин, сохранён ли пароль |
| `gp_table_ddl` / `ch_table_ddl` | схема таблицы: типы, not null, default, комментарии |
| `gp_schema_tables` / `ch_schema_tables` | таблицы схемы/базы; с `name` — только похожие имена (исправление опечаток) |
| `gp_query_test` / `ch_query_test` | EXPLAIN + предупреждения (motion, объём чтения), данные не читает, DML/DDL не принимает |
| `gp_query_run` / `ch_query_run` | то же + первые строки результата в read-only режиме. **Требует подтверждения** (см. ниже) |

## Ограничения и меры предосторожности

- ⚠️ `gp_query_run` и `ch_query_run` отдают данные из DWH в модель. **Запрашиваемые
  данные не должны быть персональными** — ФИО, телефоны, email, адреса, паспортные
  и платёжные данные, идентификаторы клиентов и сотрудников. Перед подтверждением
  вызова смотри на список колонок в запросе и отклоняй, если там есть ПДн.
  Полезно найти страницу источника данных в Confluence — там есть статус ПДн каждой колонки.
- В разрешениях `permission` стоят `"dwh_gp_query_run": "ask"` и `"dwh_ch_query_run": "ask"`.
  Не меняй их на `allow`: подтверждение — место, где ты проверяешь запрос на ПДн.
- MCP-сервер Atlassian запускается с `READ_ONLY_MODE: "true"` — записи в Jira/Confluence недоступны.
- Контуры DWH в `connections.yaml` — readonly: изменения данных на уровне БД невозможны.
- Пути к MCP-серверам в `command` — только абсолютные (см. выше).
- Обновление: `git pull` в root + `git submodule update --recursive`; в skills —
  `git pull` + `uv sync`.

## Добавление инструмента в dwh-mcp

Логика — отдельным модулем в `src/dwh_mcp/`, функция возвращает строку и
бросает `DwhError` с понятным текстом. Регистрация — `@mcp.tool()` в `server.py`.
Docstring инструмента — единственное, что модель видит постоянно, поэтому пиши
в нём и что инструмент делает, и когда его применять.
