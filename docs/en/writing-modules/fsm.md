# FSM — State Machine

**What's this?** FSM lets you have a step-by-step conversation with the user.

**When to use:**
- You need to ask for multiple values sequentially
- Registration form, survey, settings
- Any multi-step logic

---

## StatesGroup and State

First, define the states:

```python
from mxc.fsm import StatesGroup, State

class AskStates(StatesGroup):
    name = State()    # step 1: ask name
    age = State()     # step 2: ask age
    city = State()    # step 3: ask city
```

Each `State()` is a separate dialog step.

---

## FSMContext

The state handler receives `ctx` — the context:

| Method | What it does |
|--------|--------------|
| `await ctx.update_data(key=val)` | Update session data |
| `await ctx.get_data()` | Get all data as dict |
| `await ctx.set_state(NextState)` | Move to the next step |
| `await ctx.clear()` | Reset state and data |
| `await ctx.finish()` | Reset only state (keep data) |

---

## Simplest Example — WITHOUT Pydantic

```python
from mxc.fsm import StatesGroup, State
from mxc import utils
from .. import loader


class FormStates(StatesGroup):
    name = State()
    age = State()


@loader.tds
class FormModule(loader.Module):
    strings = {
        "ask_name": "What's your name?",
        "ask_age": "Nice to meet you, <b>{name}</b>! How old are you?",
        "done": (
            "✅ Done!<br><br>"
            "Name: <code>{name}</code><br>"
            "Age: <code>{age}</code>"
        ),
    }

    @loader.command()
    async def form(self, mx, event):
        """Start the survey"""
        mx.fsm.set_state(event, FormStates.name)
        await utils.answer(mx, self.strings["ask_name"], event=event)

    @loader.state(FormStates.name)
    async def got_name(self, mx, event, ctx):
        """User entered their name"""
        name = event.content.body.strip()
        
        await ctx.update_data(name=name)
        await ctx.set_state(FormStates.age)
        
        await utils.answer(
            mx,
            self.strings["ask_age"].format(name=name),
            event=event,
        )

    @loader.state(FormStates.age)
    async def got_age(self, mx, event, ctx):
        """User entered their age"""
        age = event.content.body.strip()
        
        await ctx.update_data(age=age)
        data = await ctx.get_data()
        await ctx.clear()
        
        await utils.answer(
            mx,
            self.strings["done"].format(**data),
            event=event,
        )
```

---

## How It Works Step by Step

| Step | What happens |
|------|--------------|
| 1 | User types `.form` |
| 2 | Command calls `mx.fsm.set_state(event, FormStates.name)` |
| 3 | Bot asks: "What's your name?" |
| 4 | User replies: `Mike` — **NOT a command, no dot** |
| 5 | Bot sees there's an active state `FormStates.name` |
| 6 | Calls handler `@loader.state(FormStates.name)` |
| 7 | Handler saves the name via `ctx.update_data()` |
| 8 | Moves to next state: `ctx.set_state(FormStates.age)` |
| 9 | And so on... |
| 10 | At the end: `ctx.clear()` to reset the state |

---

## Validation — Manual

If you need to check that the user entered a number:

```python
@loader.state(FormStates.age)
async def got_age(self, mx, event, ctx):
    text = event.content.body.strip()
    
    if not text.isdigit():
        await utils.answer(mx, "❌ Enter a number!", event=event)
        return  # DON'T call set_state — stay in the same state
    
    age = int(text)
    if age < 1 or age > 150:
        await utils.answer(mx, "❌ That's not a real age!", event=event)
        return
    
    await ctx.update_data(age=age)
    # ... continue
```

**If the user makes a mistake** — just don't call `ctx.set_state()` and the bot stays in the same state.

---

## Commands Interrupt FSM

If the user types **any command** (with a dot) in the middle of a dialog:
```
.help
```

The state **automatically resets**. This is a feature, not a bug.

---

## More Examples

### Example 1: Yes/No Confirmation

```python
class ConfirmStates(StatesGroup):
    confirm = State()


@loader.state(ConfirmStates.confirm)
async def got_confirm(self, mx, event, ctx):
    text = event.content.body.strip().lower()
    
    if text in ["yes", "y"]:
        # do something
        await utils.answer(mx, "✅ Confirmed!", event=event)
        await ctx.clear()
    elif text in ["no", "n"]:
        await utils.answer(mx, "❌ Cancelled", event=event)
        await ctx.clear()
    else:
        await utils.answer(mx, "Enter yes or no", event=event)
        # stay in the same state
```

### Example 2: Skipping a Step

```python
class OptionalStates(StatesGroup):
    name = State()
    bio = State()


@loader.state(OptionalStates.bio)
async def got_bio(self, mx, event, ctx):
    text = event.content.body.strip()
    
    if text == "-" or text == "skip":
        # user wants to skip
        await ctx.update_data(bio="not specified")
    else:
        await ctx.update_data(bio=text)
    
    # ... continue
```

### Example 3: ctx.data — storing intermediate data

```python
@loader.state(FormStates.name)
async def step1(self, mx, event, ctx):
    name = event.content.body.strip()
    await ctx.update_data(name=name, step=1)
    await ctx.set_state(FormStates.next)


@loader.state(FormStates.next)
async def step2(self, mx, event, ctx):
    data = await ctx.get_data()
    # data = {"name": "Mike", "step": 1}
```

---

## FSM — Under the Hood

States are stored like this:
```python
{
    "!room:server:@user:server": {
        "state": "FormStates:name",
        "data": {"name": "Mike"},
    }
}
```

Key = `room_id:user_id`. One user can only be in one state at a time.

---

## Pydantic — if you want automatic validation

If you don't want to manually check `isdigit()` and such, you can use Pydantic. This is optional.

```python
from pydantic import BaseModel, Field, model_validator, ConfigDict


class AgePayload(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True)
    age: int = Field(ge=1, le=150)

    @model_validator(mode='before')
    @classmethod
    def parse(cls, v):
        if isinstance(v, str):
            try:
                return {"age": int(v.strip())}
            except ValueError:
                raise ValueError("Enter a number!")
        return v


@loader.state(FormStates.age)
async def got_age(self, mx, event, ctx, payload: AgePayload):
    # payload.age is already int and already validated
    await ctx.update_data(age=payload.age)
    # ...
```

If validation fails — the bot sends an error and **stays in the same state**.

---

**For regular folks:** What FSM is in simple terms — see [Key Concepts](../key-concepts.md#fsm-step-by-step-dialogs).
