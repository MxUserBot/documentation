# HelperModule — Центр помощи и управления

**Файл:** `modules/help.py`

Центральный модуль справки. Предоставляет список всех активных модулей, подробную информацию по каждому модулю (команды, конфиг, описание), а также управление конфигурацией модулей на лету.

## Команды

### `.help [module]`


Без аргументов — выводит список всех активных модулей, разбитый на страницы по 5 штук. Навигация через эмодзи-кнопки ⬅️ ➡️.

```
.help
# → 💠 | MxUserBot
#    Helper Center
#
#    Available modules (Page 1/3):
#
#    ▫️ HelperModule
#    Helper Centre
#    ⬥ help, info, cfg
#
#    ▫️ LoaderModule
#    Module Manager
#    ⬥ mdl, msearch, addrepo, ...
#
#    [⬅️] [➡️]
```

С именем модуля — подробная информация:
- Название и описание
- Секция конфига (если есть) с текущими значениями
- Список команд с префиксом и описанием

```
.help pingpong
# → 📦 | PingPong
#    ℹ️ | Description: Simple ping-pong + dm checker
#
#    🛠 | Commands:
#     • .ping — Check bot latency
```

Если модуль не найден — выводит сообщение об ошибке с поиском по name и по class name.

### `.info`

**Доступ:** EVERYONE

Отправляет системную информационную карточку в виде изображения с наложенным текстом:
- Версия юзербота
- Ссылка на репозиторий

```
.info
# → (изображение с version и ссылкой)
```


### `.cfg <module> <key> <value>`

**Доступ:** EVERYONE

Изменяет конфигурационный параметр модуля. Ищет модуль по ключу `active_modules`, затем по `Meta.name`. Проверяет наличие `config.set()`.

```
.cfg LoaderModule repo_warn_ok true
.cfg TranslateMaster api_key abc123
```

При успехе: `✅ | Config Updated: <key> for <mod> set to <val>`
При ошибке: `❌ | Config Update Failed: Invalid key or validation error for <key>.`