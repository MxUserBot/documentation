# Commands and Argument Parsing

## Command Signature

```python
async def cmd(self, mx, event):
```

**required**:
- `self` — module
- `mx` — MXBotInterface (client, security, fsm, ...)
- `event` — MessageEvent

## Decorator

```python
@loader.command()
@loader.command(name="hi")
@loader.command(aliases=["hello", "hi"])
@loader.command(security=loader.EVERYONE)
```

## 3 Ways to Parse Arguments

### Type 1: Simple Typing

Just add the variables you need as arguments.
`argument: type = default`
```python
@loader.command()
async def tr(self, mx, event, lang: str = "en", text: str = None):
    """<lang> <text> - translate"""
    # .tr ru hello world → lang="ru", text="hello world"
```
If an argument with `None` default stays `None` (user didn't provide it) — the bot will show the command help.

For example `.tr` without arguments: `lang="en"` (has default), `text=None` (no argument) → shows help.

But if the user replied to someone's message with `.tr` — the bot can grab text from the reply instead of an argument. This works automatically.

### Type 2: Pydantic BaseModel

```python
from pydantic import BaseModel, Field, model_validator, ConfigDict

class SplitPayload(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True)
    top: str = Field(min_length=1)
    bottom: str = Field(default="")

    @model_validator(mode='before')
    @classmethod
    def parse(cls, v):
        if isinstance(v, str):
            parts = v.split("|", 1)
            return {"top": parts[0], "bottom": parts[1] if len(parts) > 1 else ""}
        return v


@loader.command()
async def cmd(self, mx, event, payload: SplitPayload):
    """<top> | <bottom> - demo"""
    # .cmd hello | world → payload.top="hello", payload.bottom="world"
```

**What it gives you:**
- Auto-validation (min_length, ge, regex)
- Flexible parsing (split, regex, whatever)

### Type 3: Manual Parsing

```python
@loader.command()
async def rp(self, mx, event):
    """<action> [@user] - RP command"""
    raw = await utils.get_args_raw(mx, event)
    parts = raw.split(maxsplit=2)
    subcmd = parts[0].lower() if parts else ""
    # total freedom
```

### When to use what?

Up to you — use whatever you prefer. Don't like Pydantic? Use manual parsing or function arguments.

---

## config

```python
config = {
    "api_key": loader.ConfigValue(
        default=None, description="API key", required=True,
    ),
    "delay": loader.ConfigValue(
        default=5, description="Delay (sec)",
        validator=lambda x: isinstance(x, int) and x >= 0,
    ),
    "secret": loader.ConfigValue(
        default="", description="Secret", forbid=True,
    ),
}
```

- `required=True` — won't let anyone use commands until it's filled in. **Make sure to set `default=None`**, not an empty string.
- `forbid=True` — prevent changes.
- `validator` — return True/False

Access: `self.config.get("key")` / `self.config["key"]`
Set: `self.config.set("key", value)` / `await self.config.set_async("key", value)`

---
