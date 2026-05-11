# Commands and Argument Parsing

## Command signature

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

## 3 ways of argument parsing

### Type 1: Simple type hints

Put the variables you need as arguments.
argument: type = something
```python
@loader.command()
async def tr(self, mx, event, lang: str = "en", text: str = None):
    """<lang> <text> - translate"""
    # .tr ru hello world → lang="ru", text="hello world"
```
If the default is None and the function doesn't replace None with anything, the function's help text will be shown.

For example if we type .tr
the text variable stays None.

At the same time, we can get the text from a reply to the message.

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

**Benefits:**
- Auto-validation (min_length, ge, regex)
- Flexible parsing (split, regex, anything)

### Type 3: Manual parsing

```python
@loader.command()
async def rp(self, mx, event):
    """<action> [@user] - RP command"""
    raw = await utils.get_args_raw(mx, event)
    parts = raw.split(maxsplit=2)
    subcmd = parts[0].lower() if parts else ""
    # full freedom
```

### When to use what?

It depends on the situation — use whatever you prefer. Don't like Pydantic? Use manual parsing or function arguments.

---

## config

```python
config = {
    "api_key": loader.ConfigValue(
        default="", description="API key", required=True,
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

- `required=True` — prevents using commands until filled. Note that default should be None in that case.
- `forbid=True` — prevent changing.
- `validator` — return True/False

Access: `self.config.get("key")` / `self.config["key"]`
Setting: `self.config.set("key", value)` / `await self.config.set_async("key", value)`

---
