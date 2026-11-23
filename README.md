#work-opencode-config
Конфинг для корпоративного агента для работы с набором скиллов
 
# Настройка
## exports
### Путь к конфигу:
```sh
export OPENCODE_CONFIG_DIR=~/Desktop/work-opencode-setup/work-opencode-config # Без слеша в конце! 
export OPENCODE_SETUP_DIR=~/Desktop/work-opencode-setup # Без слеша в конце! 
```

### Подключения к бд
В папке `work-opencode-setup/work-opencode-skills/`
Требуется настроить (исправить):
1. Хосты, порты (`connections.yaml`), если содержатся устаревшие
2. Логин (не пароль!!) (`connections.local.yaml`), пример в example-версии
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
