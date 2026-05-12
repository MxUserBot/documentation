# SecurityModule — Управление доступом

**Файл:** `src/mxuserbot/modules/sudo.py`
**Класс:** `SecurityModule`
**Теги:** `settings`

## Описание

Модуль управления правами доступа. Позволяет выдавать SUDO-права другим пользователям, открывать доступ к отдельным модулям или командам, а также выдавать временный доступ к командам на определённое время.

## Система уровней доступа

- **OWNER** — владелец бота (полный доступ ко всему)
- **SUDO** — доверенные пользователи (доступ к командам с уровнем `SUDO`)
- **EVERYONE** — все пользователи (команды общего доступа, например `.help`)
- **mod_perms** — точечный доступ к модулям/командам
- **tsec** — временный доступ к командам по времени

## Команды

### `.sudo add/rm/list @user:server`

**Доступ:** OWNER

Управление списком SUDO-пользователей.

```
.sudo add @friend:matrix.org
# → 👤 User @friend:matrix.org is now SUDO.

.sudo list
# → 👤 SUDO users:
#    • @friend:matrix.org

.sudo rm @friend:matrix.org
# → 👤 User @friend:matrix.org is no longer SUDO.
```

**Авто-извлечение MXID:** Если ответить на сообщение пользователя, бот автоматически извлечёт MXID из `formatted_body` (парсит `href`). Это позволяет добавлять в SUDO без ручного ввода MXID.

**Валидация MXID:** Проверяет соответствие регулярному выражению `^@.+:.+$`.

### `.modaccess add/rm @user:server <name>`

**Доступ:** OWNER

Точечное открытие доступа к конкретному модулю (по имени класса) или команде (по имени).

```
.modaccess add @user:server PingPongModule   # весь модуль PingPong
.modaccess add @user:server ping              # только команда .ping
.modaccess rm @user:server ping               # отозвать доступ
```

Проверяет, существует ли указанный модуль или команда, перед применением.

### `.tsec @user:server <command> <minutes>`

**Доступ:** OWNER

Временный доступ к команде на указанное количество минут. После истечения времени бот автоматически отправит уведомление в чат, где был выдан доступ.

```
.tsec @guest:server ping 5
# → ⏱ @guest:server now has 5 min. for command ping
```

По истечении времени:
```
⏰ | guest, ваше время истекло. теперь вы не можете юзать команду ping
```

## Структуры данных

### mod_perms
```python
{
    "@user:server": ["PingPongModule", "ping", ...]
}
```
Хранится в БД: `core.mod_perms`

### tsec_users
```python
[
    {
        "target": "@user:server",
        "command": "ping",
        "expires": 1234567890.0,
        "room_id": "!room:server"
    }
]
```
Хранится в БД: `core.tsec_users`

## Особенности

- Проверка доступа происходит в `core/security.py` через `check_access()`
- Время жизни tsec проверяется при каждом вызове команды
- Просроченные tsec автоматически удаляются из списка
- Все изменения сразу сохраняются в БД
- Работает в связке с `core/security.py` и `core/callback.py`
