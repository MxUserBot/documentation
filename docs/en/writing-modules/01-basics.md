# Basics

## Minimal module

```python
from mxc import utils
from .. import loader


class Meta:
    name = "HelloWorld"
    description = "my first module"
    version = "1.0.0"
    tags = ["utility"]


@loader.tds
class HelloWorldModule(loader.Module):
    strings = {
        "hello": "Hello, world!",
    }

    @loader.command()
    async def hello(self, mx, event):
        """Help text"""
        await utils.answer(mx, self.strings["hello"], event=event)
```

---


### How it works

When the module loads, `@loader.tds`:
1. Checks that `strings` exists on the class
2. Collects all `<name>_doc` attributes from module methods
3. Adds them to `strings` under `<mark><obj>` keys

### Format

```python
strings = {
    "greeting": "Hello, <b>{name}</b>!",
    "result": "Answer: <code>{value}</code>",
    "error": "❌ Error: {e}",
}
```

Supports **HTML** — you can use `<b>`, `<code>`, `<i>`, `<br>`.

### Usage

```python
await utils.answer(mx, self.strings["greeting"].format(name=username), event=event)
await utils.answer(mx, self.strings["result"].format(value=42), event=event)
```

### If strings is omitted

```python
# ❌ Module will NOT load:
@loader.tds
class BrokenModule(loader.Module):
    # no strings = {}
    ...
```

The core will raise an error when loading the module.

---

## File structure

```
module.py
├── class Meta:
│   ├── name              # required
│   ├── description       # required
│   ├── version           # required
│   ├── tags              # required
│   │
│   ├── author            # optional
│   └── dependencies      # optional
│
└── class XxxModule(loader.Module):
    ├── strings = {}         # REQUIRED
    ├── config = {}
    ├── async def _matrix_start(self, mx):  # start [optional]
    ├── def _matrix_stop(self, mx):         # stop [optional]
    ├── @loader.command()                   # command
    ├── @loader.watcher("regex")            # auto-reaction
    ├── @loader.on(EventType.XXX)           # event
    ├── @loader.state(SomeState)            # FSM
    └── @loader.cron("30m")                 # periodic task
```

---

## Meta

REQUIRED — create a Meta class:
```python
class Meta:
    name = "TranslateMaster"
    description = "translates text"
    version = "2.1.0"
    tags = ["utility", "translate"]
    author = "@durov:matrix.org"
    dependencies = ["googletrans==4.0.0-rc1"]
```
If you don't write Meta, the core simply won't recognize the module and won't load it.
Required fields: `name`, `description`, `version`, `tags`.

---
## strings — REQUIRED

`strings = {}` — dictionary of all module texts. **The module will not start without it.**

Every command and every handler can have its own key in `strings`. `@loader.tds` checks for `strings` at load time — if it's missing, the class will not be registered.


## @loader.tds

Class decorator. **Required.** Without it the module will NOT start.

What it does:
1. Checks that `strings = {}` exists on the class
2. Collects `*_doc` from methods into `strings`
3. Replaces `__doc__` for i18n
4. Registers the module in the core

---

## Lifecycle

```python
async def _matrix_start(self, mx):
    """Module start — async initialization"""
    self._data = await utils.request(
        "https://api.example.com/data", return_type="json",
    )
    self._bg_task = asyncio.create_task(self._loop())


def _matrix_stop(self, mx):
    """Module stop — resource cleanup"""
    self._bg_task.cancel()
```

- `__init__` is not async. For async initialization use `_matrix_start`
- `_matrix_stop` can be sync or async

---

## Debug

```python
self.log.info("Working")
self.log.error(f"Error: {e}")
```

WARNING+ logs go to the `[LOGS]` room.

---

## Exceptions

```python
from mxc.exceptions import UsageError


@loader.command()
async def cmd(self, mx, event, payload: str):
    if not payload:
        raise UsageError("Need text!")
```

---

## Important points

1. Meta — required (name, description, version, tags)
2. `strings = {}` — REQUIRED, without it the module won't load
3. `@loader.tds` — required
4. Class name contains "Module", e.g. `MikuModule`
5. Don't use `event.reply` — only `utils.answer(mx, text, event=event)`
6. `__init__` is not async. For async initialization use `_matrix_start`
7. Community modules get ScopedDatabase — only their own keys
8. Don't touch sys/subprocess/socket — the firewall won't allow it
