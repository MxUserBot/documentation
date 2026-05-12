# SECURITY: СИСТЕМА БЕЗОПАСНОСТИ

## Уровни доступа

```python
OWNER     = 1  # бит 0
SUDO      = 2  # бит 1
EVERYONE  = 4  # бит 2
```

По умолчанию всё выполняемые команды, могут выполнять — **SUDO**.

### Как задать?

```python
@loader.command()                             # SUDO
@loader.command(security=loader.OWNER)        # только OWNER
@loader.command(security=loader.EVERYONE)     # все
```

### OWNER
- Сам бот (whoami) — автоматически
- Добавляется: `.sudo add @user:server`
- **Может всё.** Проходит любые проверки.

### SUDOS
- `.sudo add @user:server`
- Могут SUDO + EVERYONE команды
- **НЕ** могут OWNER команды

### EVERYONE
- Любой в комнате
- Для публичных команд

### ModAccess
```python
modaccess add @user:module_name
```
Даёт доступ к конкретному модулю/команде без sudo.

### TSEC (временный)
```python
tsec @user:server command 300
```
Истекает через N секунд.

## check_access — цепочка проверок

```
1. Юзер в owners?     → ПРОПУСК
2. Юзер в sudos?      → ПРОПУСК
3. Команда EVERYONE?  → ПРОПУСК
4. Команда SUDO+sudo? → ПРОПУСК
5. Есть mod_perms?    → проверка
6. Команда в mod_perms?  → ПРОПУСК
7. Класс в mod_perms?    → ПРОПУСК
8. Есть tsec?            → ПРОПУСК
9. → ОТКАЗ
```

## Файрвол (audit hook)

Бот вешает `sys.addaudithook()` при старте:

### AST-анализ кода
Перед выполнением community модуля код проверяется:
- **Запрещённые атрибуты:** `crypto`, `crypto_enabled`, `device_id`
- **Запрещённые импорты:** `sys`, `subprocess`, `ctypes`, `importlib`, `shutil`, `socket`, `pty`, `builtins`
- **Запрещённые функции:** `eval()`, `exec()`, `__import__()`

### Блокировка во время работы

| Операция | Блокировка |
|----------|-----------|
| `open()` с `w/a/x/+` | Запись из community кода |
| `open()` на core файлы | Доступ к `/core/` |
| `os.remove()` | Удаление core файлов |
| `os.rename()` | Переименование core |
| `import` core | Импорт core из community |
| `ctypes.*` | Доступ к памяти |

### Замороженные core модули

```python
# Community код не может изменить core модуль:
object.__setattr__(core_module, "attr", "val")
# → PermissionError: Core modules are frozen!
```

## ScopedDatabase

Community модули получают `ScopedDatabase`:
- Все ключи с префиксом `module_name:`
- Не могут читать/писать чужие данные

## Как сделать команду только для OWNER?

```python
# Через security
@loader.command(security=loader.OWNER)
async def admin_cmd(self, mx, event):
    ...

```

## Кастомная система

```python
config = {
    "access": loader.ConfigValue(
        default="all",
        description="all, sudo, owner, или список mxid через запятую",
    ),
}

def _has_access(self, mx, sender):
    conf = self.config.get("access", "all").strip().lower()

    if conf == "all":
        return True
    if conf == "owner":
        return mx.security.is_owner(sender)
    if conf == "sudo":
        return (
            sender in mx.security.sudos
            or mx.security.is_owner(sender)
        )

    allowed = [
        x.strip()
        for x in conf.replace("\n", ",").split(",")
        if x.strip()
    ]
    return sender in allowed
```

## Gate — защита event handler'ов

```python
self.client.add_event_handler(
    EventType.ROOM_MEMBER,
    self.security.gate(cb.invite_cb),
)
```

Если отправитель не OWNER/SUDO — ивент игнорируется.


Нужно понимать, что Security тут не идеален, не защитит от всего, так что ответственность полностью на вас и пользователях.
