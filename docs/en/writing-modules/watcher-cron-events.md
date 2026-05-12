# Watcher, Cron, Event Handler

## Watcher — message observer

**What's this?** A Watcher reacts to ANY message matching a regex pattern, not just commands with a prefix.

**Important:** Watchers run ONLY if:
- The message is NOT a command (doesn't start with `.`)
- The message is NOT being processed by FSM

**Processing order:**
1. Commands (`.ping`)
2. FSM (if active)
3. Watchers

---

## Watcher Signature

```python
@loader.watcher(regex)
async def handler(self, mx, event, match):
```

- `regex` — a regular expression string
- `match` — the regex match result

---

## Examples

### Auto-reaction to "thanks"

```python
@loader.watcher(r"(?:^|\s)(?:thx|thanks|thank you)(?:$|\s)")
async def auto_thanks(self, mx, event, match):
    await event.react("❤️")
```

### Auto-currency conversion

```python
import re

@loader.watcher(r"(\d+(?:\.\d+)?)\s*(\$|€|£|rub|usd|eur)", flags=re.IGNORECASE)
async def auto_convert(self, mx, event, match):
    amount = match.group(1)
    currency = match.group(2).lower()
    # ... conversion
```

---

## About `match`

What gets passed in `match`:
- If the regex has named groups `(?P<name>...)` → `match.groupdict()`
- If the regex has ordinary groups → `match.groups()` or `match.group(1)`
- If no groups → `match.group(0)`

### With named groups

```python
@loader.watcher(r"convert (?P<amount>\d+) (?P<currency>\w+)")
async def converter(self, mx, event, match: dict):
    # match = {"amount": "100", "currency": "usd"}
    amount = match["amount"]
    currency = match["currency"]
```

### Without groups

```python
@loader.watcher(r"hello")
async def on_hello(self, mx, event, match: str):
    # match = "hello"
    await utils.answer(mx, "Hi there!", event=event)
```

---

## Watcher Details

- **All watchers run** if their regex matches. If 5 modules have a watcher for the same thing — all 5 will run.
- **Async**: Run via `asyncio.create_task()`
- **Do NOT block** other watchers or commands
- **No order**: No guarantee which watcher runs first

---

## Event Handler — any Matrix events

Reacts not only to messages but to ANY Matrix events:

```python
from mautrix.types import EventType

@loader.on(EventType.ROOM_MEMBER)
async def on_member(self, mx, event):
    """Someone joined/left the room"""
    if event.content.membership == "join":
        await utils.answer(mx, "Welcome!", event=event)
```

**Common EventType:**
- `EventType.ROOM_MESSAGE` — new message
- `EventType.ROOM_MEMBER` — join/leave/invite
- `EventType.ROOM_NAME` — room name changed
- `EventType.ROOM_TOPIC` — topic changed
- `EventType.REACTION` — reaction added
- `EventType.ROOM_ENCRYPTED` — encrypted message

---

## Cron — scheduled tasks

Runs automatically on a schedule. Doesn't depend on messages.

```python
@loader.cron("30m")
async def every_half_hour(self, mx):
    """Every 30 minutes"""
    self.log.info("30 minutes passed!")
```

**Supported formats:**

| Format | Example | Interval |
|--------|---------|----------|
| `N s` | `60s` | 60 seconds |
| `N m` | `30m` | 30 minutes |
| `N h` | `2h` | 2 hours |
| Cron expression | `*/5 * * * *` | every 5 minutes |
| Cron expression | `0 * * * *` | every hour |

**Fallback:** If the format is unrecognized — uses `60s`.

---

## Cron Examples

### Every 5 minutes (cron)

```python
@loader.cron("*/5 * * * *")
async def check_status(self, mx):
    """Check status every 5 minutes"""
    try:
        data = await utils.request("https://api.example.com/status")
        self.log.info(f"Status: {data}")
    except Exception as e:
        self.log.error(f"Check failed: {e}")
```

### Hourly backup

```python
@loader.cron("1h")
async def hourly_backup(self, mx):
    """Every hour"""
    # ... do backup
```

---

## Cron Details

- **Runs on start**: First execution happens immediately when the module loads
- **Stops on unload**: When the module is removed — cron stops
- **Parameters**: Only `(self, mx)` — no `event`
- **No overlap**: If a task runs longer than its interval, the next one still runs on schedule

---

**For regular folks:** Simple explanations of these concepts — see [Key Concepts](../key-concepts.md).
