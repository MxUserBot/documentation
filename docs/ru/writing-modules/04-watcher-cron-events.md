# Watcher, Cron, Event Handler

## Watcher

Срабатывает на каждое сообщение, если regex совпадает.

```python
@loader.watcher(r"(?:^|\s)(?:\$|\b(?:usd|eur|rub)\b)", security=loader.EVERYONE)
async def auto_convert(self, mx, event, match: str):
    """Конвертация валют из любого сообщения"""
```

- `match` — объект `re.Match` от regex
- Запускается после команд, если сообщение не FSM и не команда
- Все watcher'ы всех модулей проверяются подряд

Подробнее: `../0-key-concepts.md` → раздел Watcher'ы

---

## Event Handler

Обработчик произвольного события Matrix.

```python
from mautrix.types import EventType


@loader.on(EventType.ROOM_MEMBER)
async def on_member(self, mx, event):
    if event.content.membership == "join":
        await utils.answer(mx, "Привет!", event=event)
```

---

## Cron — периодические задачи

Выполняется с фиксированным интервалом, не зависит от сообщений.

```python
@loader.cron("30m")
async def every_half_hour(self, mx):
    """Будет вызываться каждые 30 минут"""
    await utils.answer(mx, "🕐 Прошло 30 минут!", room_id="!room:server")


@loader.cron("*/5 * * * *")
async def every_five_minutes(self, mx):
    """Cron-выражение: каждая 5-я минута"""
    ...


@loader.cron("2h")
async def every_two_hours(self, mx):
    ...
```

Поддерживает:
- **Интервалы**: `60s`, `30m`, `2h` (секунды, минуты, часы)
- **Cron-выражения**: `*/5 * * * *`, `0 * * * *`

Метод cron принимает `(self, mx)`. Запускается при старте модуля, останавливается при выгрузке.

Подробнее: `../0-key-concepts.md` → раздел Cron
