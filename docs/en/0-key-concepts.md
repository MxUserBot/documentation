---

## Decorators

### `@loader.command(name=None, aliases=[], security=SUDO)`

Registers a module method as a command. The command name is the function name (or `name`), aliases are alternative names. Default access level is `SUDO`.

```python
@loader.command()
async def ping(self, mx, event):
    """Check bot latency"""
    ...

@loader.command(aliases=["df"], security=loader.OWNER)
async def disk_free(self, mx, event):
    ...
```

### `@loader.watcher(regex, security=EVERYONE)`

Registers a watcher — the function runs on EVERY message if it matches the regex. Compiled with `re.IGNORECASE`. Runs **after** commands, if the message was not handled by FSM and is not a command.

```python
@loader.watcher(r"(?:спс|спасибо|thx|thanks)", security=loader.EVERYONE)
async def auto_thanks(self, mx, event, match):
    await event.react("❤️")
```

**Process:** all watchers from all modules are checked in `message_cb`.
If the message matches FSM or a command — watchers do not run.

Details: [Watchers](#watchers)

### `@loader.on(event_type)`

Registers a handler for an arbitrary Matrix event. The event type can be any `EventType`. Handlers are called in `_dispatch_event()`.

```python
@loader.on(EventType.ROOM_NAME)
async def on_room_rename(self, mx, event):
    await utils.answer(mx, f"Room renamed!", event)
```

### `@loader.state(State)`

Registers an FSM state handler. Called when a user in state `State` sends a message (without a prefix).

```python
@loader.state(AskStates.name)
async def ask_name(self, mx, event, ctx: FSMContext):
    name = event.content.body.strip()
    await ctx.update_data(name=name)
    await ctx.set_state(AskStates.age)
```

**Process:** in `message_cb`, `fsm.get_state()` is checked first. If a state is active — a handler with `@loader.state(<current_state>)` is looked up. If found — it runs with `FSMContext` as an additional argument. If the message has a prefix — the state is reset (finish).

Details: [FSM](#fsm-finite-state-machine)

### `@loader.cron("30m")` / `@loader.cron("*/5 * * * *")`

Registers a periodic task. The interval is a string with a suffix (`60s`, `30m`, `2h`) or a cron expression (`*/5 * * * *` — every 5 minutes).

```python
@loader.cron("30m")
async def auto_status(self, mx):
    await mx.client.set_status("online")
```

Details: [Cron](#cron)

---

## FSM (Finite State Machine)

**File:** `mxc/src/mxc/fsm.py`

A state machine for dialogs with the user. Allows a module to ask questions and receive answers sequentially.

### Structure

```python
class FSM:
    _states = {
        "!room:server:@user:server": {
            "state": "AskStates:name",
            "data": {"name": "Alice"},
            "_expires_at": 1234567890.0  # if ttl > 0
        }
    }
    _processed_events: set[str]
```

---

## Watchers

Watchers are handlers that react to message text via regex. They run **only** if the message did not pass through FSM and is not a command.

### How they work

```python
@loader.watcher(r"hello", security=EVERYONE)
async def on_hello(self, mx, event, match):
    ...
```

- All watchers from all modules are checked in sequence
- If multiple modules have a watcher for the same regex — all of them run
- Runs asynchronously via `asyncio.create_task`
- `match` is passed as a keyword argument (can be `match.groupdict()`, a string from groups, or `match.group(0)`)

---

## Cron

Periodic tasks that run at a fixed interval. Do not depend on messages.

### Interval parsing

`_parse_cron(s)` in `core/loader.py:123`:

| Format | Example | Result |
|--------|---------|--------|
| `N s/m/h` | `30m` | 1800s |
| `*/N * * * *` | `*/5 * * * *` | 300s |
| `0 * * * *` | `0 * * * *` | 3600s |
| other | — | 60s (fallback) |

---

## Rate Limiter (AIMD)

Protection against 429 (M_LIMIT_EXCEEDED) from the Matrix server. Implements AIMD (Additive Increase Multiplicative Decrease) — an adaptive algorithm for controlling request rate.

### Enabling

In `__main__.py` when creating `MXCClient`:
```python
self.client = MXCClient(
    ...,
    rate_limit_protect=True  # → mautrix_rate_limit_patch()
)
```

A global patch intercepts `HTTPAPI._send` in mautrix and wraps all non-GET requests in the rate limiter.

### GlobalRateLimiter

**Parameters:**
| Parameter | Start | Minimum | Maximum |
|-----------|-------|---------|---------|
| `min_interval` | 0.5s | 0.4s | 15.0s |

**Algorithm:**

```
acquire():
  1. If backoff is active — sleep until it ends
  2. Wait min_interval between requests
  3. Return control

report_success():
  1. If no 429 in the last 60s:
     - Counter ok_since_last_429++
     - If >= 15 successes: min_interval *= 0.95 (speed up)
  2. If 429 in 60s: reset counter

report_error() (on 429):
  1. backoff_until = now + max(min_interval * 2, recent_429_count * 3.0, up to 15s)
  2. min_interval *= 1.50 (slow down, max 15s)
```

### Retry logic

When a 429 is received, exponential backoff with jitter is performed:
```
wait = min(2.0^attempt + random(0, 1), 30.0)
```
Maximum 5 retries.

### State (state_str)

In logs:
```
interval=0.500s 429=0.0% (0/142) backoff=0.0s ok_seq=15
interval=2.531s 429=3.1% (5/161) backoff=2.1s ok_seq=0
```

- `interval` — current interval between requests
- `429` — percentage of rate limits in the last 60s
- `backoff` — remaining backoff time
- `ok_seq` — consecutive successful requests without 429

Simply put: a regular user doesn't need to worry about their server having strict limits on reactions/deletion etc. — the rate limiter prevents the user from constantly hitting 429 errors.
Of course, this may slow down the userbot slightly, since the rate limiter adds ~400ms of request delay, but what else can you do?

---
