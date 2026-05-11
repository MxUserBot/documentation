# Watcher, Cron, Event Handler

## Watcher

Fires on every message if the regex matches.

```python
@loader.watcher(r"(?:^|\s)(?:\$|\b(?:usd|eur|rub)\b)", security=loader.EVERYONE)
async def auto_convert(self, mx, event, match: str):
    """Currency conversion from any message"""
```

- `match` — `re.Match` object from the regex
- Runs after commands, if the message is not FSM and not a command
- All watchers from all modules are checked sequentially

Details: `../0-key-concepts.md` → section Watchers

---

## Event Handler

Handler for arbitrary Matrix events.

```python
from mautrix.types import EventType


@loader.on(EventType.ROOM_MEMBER)
async def on_member(self, mx, event):
    if event.content.membership == "join":
        await utils.answer(mx, "Hello!", event=event)
```

---

## Cron — periodic tasks

Runs at a fixed interval, independent of messages.

```python
@loader.cron("30m")
async def every_half_hour(self, mx):
    """Will be called every 30 minutes"""
    await utils.answer(mx, "🕐 30 minutes passed!", room_id="!room:server")


@loader.cron("*/5 * * * *")
async def every_five_minutes(self, mx):
    """Cron expression: every 5th minute"""
    ...


@loader.cron("2h")
async def every_two_hours(self, mx):
    ...
```

Supports:
- **Intervals**: `60s`, `30m`, `2h` (seconds, minutes, hours)
- **Cron expressions**: `*/5 * * * *`, `0 * * * *`

The cron method takes `(self, mx)`. It starts when the module starts and stops when it is unloaded.

Details: `../0-key-concepts.md` → section Cron
