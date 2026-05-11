# Сборка документации через MkDocs

MkDocs превращает папку с .md файлами в красивый статический сайт.

## Установка

```bash
pip install mkdocs mkdocs-material
```

## Конфиг

В корне проекта создаётся `mkdocs.yml`:

```yaml
site_name: MXUserbot Docs
theme:
  name: material
  palette:
    - scheme: default
      toggle:
        icon: material/toggle-switch-off-outline
        name: Switch to dark mode
    - scheme: slate
      toggle:
        icon: material/toggle-switch
        name: Switch to light mode
nav:
  - Главная: index.md
  - Как это работает: 2-how-it-works.md
  - Pros/Cons: 6-pros-cons.md
  - Разработка:
    - Ключевые концепции: dev/0-key-concepts.md
    - SAS верификация: dev/1-sas-verification.md
    - Эмодзи коллбеки: dev/3-emoji-callbacks.md
    - Написание модулей:
      - Введение: dev/writing-modules/00-index.md
      - База: dev/writing-modules/01-basics.md
      - Команды: dev/writing-modules/02-commands.md
      - Ответы: dev/writing-modules/03-answer.md
      - Watcher/Cron/Events: dev/writing-modules/04-watcher-cron-events.md
      - FSM: dev/writing-modules/05-fsm.md
    - Безопасность: dev/5-security.md
    - ZIP модули: dev/7-zip-modules.md
    - Справочник utils: dev/8-utils-reference.md
  - Системные модули:
    - Ping: system_modules/01-ping.md
    - Help: system_modules/02-help.md
    - Prefix: system_modules/03-set_prefix.md
    - Sudo: system_modules/04-sudo.md
    - Verif: system_modules/05-verif.md
    - Shell: system_modules/06-shell.md
    - Loader: system_modules/07-loader.md
```

## Запуск

```bash
# дев-сервер с авто-перезагрузкой
mkdocs serve

# сборка статики в site/
mkdocs build
```

## Деплой

```bash
# на GitHub Pages
mkdocs gh-deploy

# или просто залить site/ на любой хостинг
```

## Почему MkDocs + Material

- **Zero-config** по сравнению с Docusaurus — никакого React/Node
- .md файлы **как есть** — не нужно ничего менять
- Вложенные папки поддерживаются (`dev/`, `system_modules/`)
- Тёмная/светлая тема из коробки
- Полнотекстовый поиск
- Куча плагинов: диаграммы, формулы, версионирование
- Генерирует чистую статику — можно хостить где угодно
