# База: минимальный модуль

## Что ОБЯЗАТЕЛЬНО должно быть

Любой модуль ДОЛЖЕН содержать:

| № | Вещь | Где | Обязательно? |
|---|------|-----|--------------|
| 1 | `class Meta` | на уровне модуля (вне класса Module) | **ДА** |
| 2 | `name`, `description`, `version`, `tags` | внутри Meta | **ДА** |
| 3 | `@loader.tds` | над классом Module | **ДА** |
| 4 | `class XxxModule(loader.Module)` | любой класс который наследует `loader.Module` | **ДА** |
| 5 | `strings = {}` | внутри класса Module | **ДА** |

**Имя класса должно заканчиваться на `Module`**. Именно по этому бот понимает что это точка входа.

Например:
- `HelloModule` — правильно
- `WikipediaModule` — правильно
- `MyEpicModule` — правильно

Имя файла может быть любым. Но по конвенции обычно:
- Файл `hello.py` → класс `HelloModule`
- Файл `wikipedia.py` → класс `WikipediaModule`

---

## Минимальный рабочий модуль

Создай файл `hello.py`:

```python
from mxc import utils
from .. import loader


class Meta:
    name = "HelloWorld"
    description = "Привет мир, мой первый модуль"
    version = "1.0.0"
    tags = ["test"]


@loader.tds
class HelloWorldModule(loader.Module):
    strings = {
        "hello": "Привет, мир!",
    }

    @loader.command()
    async def hello(self, mx, event):
        """Сказать привет"""
        await utils.answer(mx, self.strings["hello"], event=event)
```

Всё. Это рабочий модуль.

---

## Давай разберём каждую часть

### 1. Импорты

```python
from mxc import utils
from .. import loader
```

- `utils` — функции для ответа, запросов и т.д.
- `loader` — декораторы для команд, watcher'ов и т.д.

### 2. class Meta — ОБЯЗАТЕЛЬНО

```python
class Meta:
    name = "HelloWorld"           # имя модуля
    description = "..."            # описание
    version = "1.0.0"              # версия
    tags = ["test"]                # теги для поиска
```

Meta должна быть **на уровне модуля**, не внутри класса Module. Без неё модуль просто не загрузится.

**Необязательные поля:**
```python
    author = "https://github.com/username"  # автор (ссылка на гитхаб)
    dependencies = ["aiohttp"]              # pip зависимости
```

### 3. @loader.tds — ОБЯЗАТЕЛЬНО

```python
@loader.tds
class HelloWorldModule(loader.Module):
```

Декоратор класса. Без него модуль не зарегистрируется.

Что он делает:
- Проверяет наличие `strings = {}`
- Собирает документацию команд
- Регистрирует модуль в системе

### 4. strings = {} — ОБЯЗАТЕЛЬНО

```python
strings = {
    "hello": "Привет, мир!",
}
```

**Зачем это нужно?**
- Удобно: все тексты в одном месте, а не раскиданы по коду
- Не хочешь париться — оставь `strings = {}` пустым
- В будущем: strings нужен для системы переводов (i18n)

Можно использовать HTML:
```python
strings = {
    "welcome": "Привет, <b>{name}</b>!",
    "error": "❌ Ошибка: <code>{e}</code>",
}
```

### 5. Команда

```python
@loader.command()
async def hello(self, mx, event):
    """Сказать привет"""
    await utils.answer(mx, self.strings["hello"], event=event)
```

- Имя функции = имя команды. `async def hello(...)` → команда `.hello`
- `self` — экземпляр модуля
- `mx` — интерфейс к боту (`client`, `fsm`, `security`)
- `event` — событие сообщения
- Докстрока `"""Сказать привет"""` — это хелп команды, **ОБЯЗАТЕЛЬНА**

---

## Жизненный цикл модуля

Если нужно что-то сделать при старте или остановке:

```python
@loader.tds
class MyModule(loader.Module):
    strings = {"start": "Модуль запущен!"}

    async def _matrix_start(self, mx):
        """Вызывается при загрузке модуля"""
        self.log.info("Модуль стартует")
        await utils.answer(mx, self.strings["start"], room_id="!logs:server")

    async def _matrix_stop(self, mx):
        """Вызывается при выгрузке модуля"""
        self.log.info("Модуль останавливается")
```

- `_matrix_start` — **async**, асинхронная инициализация
- `_matrix_stop` — **только async**

---

## База данных (ключ-значение)

Каждый модуль имеет доступ к БД через `self._get` и `self._set`.

```python
# Сохранить значение
await self._set("my_key", "my_value")
await self._set("counter", 42)

# Прочитать значение
value = await self._get("my_key")       # → "my_value"
counter = await self._get("counter", 0) # → 42 (default если нет)

# Удалить (если передать None — удаляется)
await self._set("my_key", None)
```

**Важно:** Community модули видят ТОЛЬКО свои ключи. ScopedDatabase автоматически меняет префикс на имя твоего модуля.

Пример счётчика:

```python
@loader.command()
async def counter(self, mx, event):
    """Счётчик вызовов"""
    count = await self._get("count", 0)
    count += 1
    await self._set("count", count)
    await utils.answer(mx, f"Счётчик: {count}", event=event)
```

---

## config — настройки модуля

Через `config` пользователь может настраивать модуль не залезая в код.

```python
config = {
    "api_key": loader.ConfigValue(
        default=None, description="API ключ", required=True,
    ),
    "delay": loader.ConfigValue(
        default=5, description="Задержка (сек)",
        validator=lambda x: isinstance(x, int) and x >= 0,
    ),
}
```

- `required=True` — не даст юзать команды пока не заполнено
- `default=None` — если required, обязательно ставь None, а не пустую строку
- `validator` — функция проверки, возвращает True/False

Доступ к config:
```python
self.config.get("api_key")        # прочитать
self.config["api_key"]            # прочитать (коротко)
self.config.set("api_key", val)   # установить
await self.config.set_async("api_key", val)  # установить (async)
```

---

## Логгирование

```python
self.log.info("Всё ок")
self.log.warning("Внимание!")
self.log.error(f"Ошибка: {e}")
```

Логи уровня WARNING и выше автоматически летят в лог-комнату.

---

## Ошибки пользователя

Если пользователь неправильно использует команду:

```python
from mxc.exceptions import UsageError

@loader.command()
async def give(self, mx, event, who: str = None):
    """<кому> — что-то дать"""
    if not who:
        raise UsageError("Нужно указать кому!")
```

Бот автоматически покажет хелп команды.

---

## Что важно запомнить

1. **`class Meta`** — обязательно, на уровне модуля
2. **`@loader.tds`** — обязательно над классом
3. **`strings = {}`** — обязательно внутри класса
4. Имя класса должно содержать `Module` в конце (`HelloModule`, `WikipediaModule`)
5. Community модули видят только свои данные в БД (ScopedDatabase)
6. Не лезь в `sys`, `subprocess`, `socket` — файрвол не пустит
