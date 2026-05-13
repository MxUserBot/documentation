# Basics: Minimal Module

## What MUST be there

Any module MUST contain:

| # | Thing | Where | Required? |
|---|-------|-------|-----------|
| 1 | `class Meta` | at module level (outside the Module class) | **YES** |
| 2 | `name`, `description`, `version`, `tags` | inside Meta | **YES** |
| 3 | `@loader.tds` | above the Module class | **YES** |
| 4 | `class XxxModule(loader.Module)` | any class inheriting `loader.Module` | **YES** |
| 5 | `strings = {}` | inside the Module class | **YES** |

**The class name must end with `Module`**. That's how the bot identifies the entry point.

For example:
- `HelloModule` — correct
- `WikipediaModule` — correct
- `MyEpicModule` — correct

The filename can be anything. But by convention:
- File `hello.py` → class `HelloModule`
- File `wikipedia.py` → class `WikipediaModule`

---

## Minimal Working Module

Create a file `hello.py`:

```python
from mxc import utils
from .. import loader


class Meta:
    name = "HelloWorld"
    description = "Hello world, my first module"
    version = "1.0.0"
    tags = ["test"]


@loader.tds
class HelloWorldModule(loader.Module):
    strings = {
        "hello": "Hello, world!",
    }

    @loader.command()
    async def hello(self, mx, event):
        """Say hello"""
        await utils.answer(mx, self.strings["hello"], event=event)
```

That's it. This is a working module.

---

## Let's Break Down Each Part

### 1. Imports

```python
from mxc import utils
from .. import loader
```

- `utils` — functions for replies, requests, etc.
- `loader` — decorators for commands, watchers, etc.

### 2. class Meta — REQUIRED

```python
class Meta:
    name = "HelloWorld"           # module name
    description = "..."            # description
    version = "1.0.0"              # version
    tags = ["test"]                # tags for searching
```

Meta must be **at module level**, not inside the Module class. Without it, the module simply won't load.

**Optional fields:**
```python
    author = "https://github.com/username"  # author (github link)
    dependencies = ["aiohttp"]              # pip dependencies
```

### 3. @loader.tds — REQUIRED

```python
@loader.tds
class HelloWorldModule(loader.Module):
```

Class decorator. Without it, the module won't be registered.

What it does:
- Checks that `strings = {}` exists
- Collects command documentation
- Registers the module in the system

### 4. strings = {} — REQUIRED

```python
strings = {
    "hello": "Hello, world!",
}
```

**Why is this needed?**
- Convenience: all text in one place, not scattered across the code
- Don't wanna bother? Leave `strings = {}` empty
- Future: strings are needed for the translation system (i18n)

You can use HTML:
```python
strings = {
    "welcome": "Hello, <b>{name}</b>!",
    "error": "❌ Error: <code>{e}</code>",
}
```

### 5. Command

```python
@loader.command()
async def hello(self, mx, event):
    """Say hello"""
    await utils.answer(mx, self.strings["hello"], event=event)
```

- Function name = command name. `async def hello(...)` → command `.hello`
- `self` — module instance
- `mx` — bot interface (`client`, `fsm`, `security`)
- `event` — message event
- Docstring `"""Say hello"""` — this is the command help, **REQUIRED**

---

## Module Lifecycle

If you need to do something on start or stop:

```python
@loader.tds
class MyModule(loader.Module):
    strings = {"start": "Module started!"}

    async def _matrix_start(self, mx):
        """Called when the module loads"""
        self.log.info("Module is starting")
        await utils.answer(mx, self.strings["start"], room_id="!logs:server")

    async def _matrix_stop(self, mx):
        """Called when the module unloads"""
        self.log.info("Module is stopping")
```

- `_matrix_start` — **async**, asynchronous initialization
- `_matrix_stop` — **async only**

---

## Database (key-value)

Every module has access to the database via `self._get` and `self._set`.

```python
# Save a value
await self._set("my_key", "my_value")
await self._set("counter", 42)

# Read a value
value = await self._get("my_key")       # → "my_value"
counter = await self._get("counter", 0) # → 42 (default if missing)

# Delete (pass None — it gets deleted)
await self._set("my_key", None)
```

**Note:** Community modules only see THEIR OWN keys. ScopedDatabase automatically prefixes keys with the module name.

Counter example:

```python
@loader.command()
async def counter(self, mx, event):
    """Command counter"""
    count = await self._get("count", 0)
    count += 1
    await self._set("count", count)
    await utils.answer(mx, f"Count: {count}", event=event)
```

---

## config — module settings

Via `config` the user can configure the module without touching the code.

```python
config = {
    "api_key": loader.ConfigValue(
        default=None, description="API key", required=True,
    ),
    "delay": loader.ConfigValue(
        default=5, description="Delay (sec)",
        validator=lambda x: isinstance(x, int) and x >= 0,
    ),
}
```

- `required=True` — won't let anyone use commands until it's filled in
- `default=None` — if required, always set None, not an empty string
- `validator` — validation function, returns True/False

Accessing config:
```python
self.config.get("api_key")        # read
self.config["api_key"]            # read (short)
self.config.set("api_key", val)   # write
await self.config.set_async("api_key", val)  # write (async)
```

---

## Logging

```python
self.log.info("All good")
self.log.warning("Watch out!")
self.log.error(f"Error: {e}")
```

Logs at WARNING level and above are automatically sent to the log room.

---

## User Errors

If a user misuses a command:

```python
from mxc.exceptions import UsageError

@loader.command()
async def give(self, mx, event, who: str = None):
    """<who> — give something"""
    if not who:
        raise UsageError("You need to specify who!")
```

The bot will automatically show the command help.

---

## What to Remember

1. **`class Meta`** — required, at module level
2. **`@loader.tds`** — required above the class
3. **`strings = {}`** — required inside the class
4. Class name must contain `Module` at the end (`HelloModule`, `WikipediaModule`)
5. Community modules only see their own data in the DB (ScopedDatabase)
6. Don't touch `sys`, `subprocess`, `socket` — the firewall won't let you
