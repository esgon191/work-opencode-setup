# Слияние скилов и конфигов
1. [x] Проверить правильность путей до opencode.jsonc и mcp — пути битые, правки описаны в разделе «Правки в сабмодулях»
2. [x] Слить .gitignore

# Документация
1. [x] Слить оба README.md в один (root README.md)
2. [x] Расписать ограничения и меры предосторожности (root README.md)

# Правки в сабмодулях (вносятся вручную в их репо)

## work-opencode-config
1. [/] `opencode.jsonc`: путь dwh-mcp `/Users/hq-fvfkjlu11wfv/Desktop/mcp/work-opencode-skills/.venv/bin/dwh-mcp` → `/Users/hq-fvfkjlu11wfv/Desktop/work-opencode-setup/work-opencode-skills/.venv/bin/dwh-mcp` (или через `{env:OPENCODE_SETUP_DIR}`, см. root README.md)
2. [ ] `opencode.jsonc`: удалить мёртвые permissions `~/Documents/opencode/skills/**` (из `permission.external_directory` и `permission.edit`)
3. [ ] `README.md`: `export OPENCODE_CONFIG_DIR=~/Desktop/work-opencode-config/` → `export OPENCODE_CONFIG=~/Desktop/work-opencode-setup/work-opencode-config/opencode.jsonc` (плюс пояснение, почему не `OPENCODE_CONFIG_DIR`)
4. [ ] `README.md`: блок env-переменных можно сократить до ссылки на перечень в root README.md
5. [ ] `TODO.md`: пункт «путь к mcp под переменную окружения» закрывается правкой №1; «перечень переменных окружения» перенесён в root README.md

## work-opencode-skills
1. [ ] `README.md`: пути `<Путь к репозиторию>/work-opencode-skills` → `<...>/work-opencode-setup/work-opencode-skills` (установка, подключение, вариант через uv)
2. [ ] `README.md`: пример `opencode.json` можно заменить ссылкой на root README.md, чтобы не дублировать

(Dev-задачи skills — новые скиллы, тесты — остаются в его собственном TODO.md)
