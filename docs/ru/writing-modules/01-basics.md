# База

## Минимальный модуль

```python
from mxc import utils
from .. import loader


class Meta:
    name = "HelloWorld"
    description = "мой первый модуль"
    version = "1.0.0"
    tags = ["utility"]


@loader.tds
class HelloWorldModule(loader.Module):
    strings = {
        "hello": "Привет, мир!",
    }

    @loader.command()
    async def hello(self, mx, event):
        """Текст хелпа"""
        await utils.answer(mx, self.strings["hello"], event=event)
```

---


### Как работает

При загрузке модуля `@loader.tds`:
1. Проверяет наличие `strings` у класса
2. Собирает все атрибуты вида `<name>_doc` у методов модуля
3. Добавляет их в `strings` под ключами `<mark><obj>`

### Формат

```python
strings = {
    "greeting": "Привет, <b>{name}</b>!",
    "result": "Ответ: <code>{value}</code>",
    "error": "❌ Ошибка: {e}",
}
```

Поддерживают **HTML** — можно использовать `<b>`, `<code>`, `<i>`, `<br>`.

### Использование

```python
await utils.answer(mx, self.strings["greeting"].format(name=username), event=event)
await utils.answer(mx, self.strings["result"].format(value=42), event=event)
```

### Если не указать strings

```python
# ❌ Модуль НЕ загрузится:
@loader.tds
class BrokenModule(loader.Module):
    # нет strings = {}
    ...
```

Ядро выдаст ошибку при загрузке модуля.

---

## Структура файла

```
module.py
├── class Meta:
│   ├── name              # обязательно
│   ├── description       # обязательно
│   ├── version           # обязательно
│   ├── tags              # обязательно
│   │
│   ├── author            # опционально
│   └── dependencies      # опционально
│
└── class XxxModule(loader.Module):
    ├── strings = {}         # ОБЯЗАТЕЛЬНО
    ├── config = {}
    ├── async def _matrix_start(self, mx):  # старт [опционально]
    ├── def _matrix_stop(self, mx):         # стоп [опционально]
    ├── @loader.command()                   # команда
    ├── @loader.watcher("regex")            # авто-реакция
    ├── @loader.on(EventType.XXX)           # ивент
    ├── @loader.state(SomeState)            # FSM
    └── @loader.cron("30m")                 # периодическая задача
```

---

## Meta

ОБЯЗАТЕЛЬНО создаем класс Meta:
```python
class Meta:
    name = "TranslateMaster"
    description = "переводит текст"
    version = "2.1.0"
    tags = ["utility", "translate"]
    author = "@durov:matrix.org"
    dependencies = ["googletrans==4.0.0-rc1"]
```
Если вы не напишете Meta — ядро просто не узнает модуль, и не загрузит его.
Обязательно: `name`, `description`, `version`, `tags`.

---
## strings — ОБЯЗАТЕЛЬНЫ

`strings = {}` — словарь всех текстовок модуля. **Без него модуль не запустится.**

Каждая команда, каждый хендлер могут иметь свой ключ в `strings`. `@loader.tds` проверяет наличие `strings` при загрузке — если его нет, класс не будет зарегистрирован.


## @loader.tds

Декоратор класса. **Обязателен.** Без него модуль стартовать НЕ будет.

Что делает:
1. Проверяет наличие `strings = {}` у класса
2. Собирает `*_doc` из методов в `strings`
3. Подменяет `__doc__` для i18n
4. Регистрирует модуль в ядре

---

## Жизненный цикл

```python
async def _matrix_start(self, mx):
    """Старт модуля — асинхронная инициализация"""
    self._data = await utils.request(
        "https://api.example.com/data", return_type="json",
    )
    self._bg_task = asyncio.create_task(self._loop())


def _matrix_stop(self, mx):
    """Остановка модуля — чистка ресурсов"""
    self._bg_task.cancel()
```

- `__init__` без async. Для асинхронной инициализации — `_matrix_start`
- `_matrix_stop` может быть sync или async

---

## Дебаг

```python
self.log.info("Работает")
self.log.error(f"Ошибка: {e}")
```

Логи WARNING+ летят в `[LOGS]` комнату.

---

## Исключения

```python
from mxc.exceptions import UsageError


@loader.command()
async def cmd(self, mx, event, payload: str):
    if not payload:
        raise UsageError("Нужен текст!")
```

---

## Что важно

1. Meta — обязательно (name, description, version, tags)
2. `strings = {}` — ОБЯЗАТЕЛЬНО, без него модуль не загрузится
3. `@loader.tds` — обязательно
4. Имя класса содержит "Module", напр. `MikuModule`
5. Не используй `event.reply` — только `utils.answer(mx, text, event=event)`
6. `__init__` без async. Для асинхронной инициализации — `_matrix_start`
7. Community модули получают ScopedDatabase — только свои ключи
8. Не лезь в sys/subprocess/socket — файрвол не пустит
