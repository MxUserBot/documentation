## How it works?

You send a message, the bot adds reactions. The user clicks a reaction → the bot triggers your callback.

```
Message: "Choose a language:"
Reactions:  🐍 🦀 ☕

User clicks 🦀
  → callback catches the reaction
  → Calls callback with payload="Rust"
```

## EmojiKeyBoard

```python
from mxc.types import EmojiButton
from mxc.utils.keyboard import EmojiKeyBoard

markup = EmojiKeyBoard(
    rows=[
        [
            EmojiButton(emoji="✅", data="yes"),
            EmojiButton(emoji="❌", data="no"),
            EmojiButton(emoji="🤔", data="maybe"),
        ],

    ],
    callback=my_handler,
)

await utils.answer(
    mx,
    "Message text",
    event=event,
    reply_markup=markup,
)
```

## All EmojiKeyBoard parameters

| Parameter | What it does | Default |
|----------|-----------|-------------|
| `rows` | Buttons in rows | **required** |
| `callback` | Your function | **required** |
| `ttl` | Time to live in seconds (0 = infinite) | 0 |
| `allowed_senders` | Who can click | sender only |
| `remove_clicked` | Remove reaction after click | True |
| `keep_reactions` | Restore if deleted | True |
| `allow_sudo` | Allow sudo to click | True |
| `data` | Session state dict | {} |

## EmojiCallbackContext

What you get in the callback:

| Field | Type | What it contains |
|------|-----|-------------|
| `ctx.payload` | Any | data from EmojiButton |
| `ctx.data` | dict | mutable session dict |
| `ctx.key` | str | Emoji key |
| `ctx.sender` | str | Who clicked |
| `ctx.message_id` | str | Message ID |
| `ctx.room_id` | str | Room ID |

Methods:
- `await ctx.edit("text")` — edit the message
- `await ctx.react("👍")` — add a reaction
- `await ctx.close()` — close + remove reactions
- `await ctx.refresh()` — restore deleted reactions

## PATTERNS

### 1. Confirm

```python
async def on_confirm(ctx):
    if ctx.payload == "yes":
        await ctx.edit("✅ Done!")
    else:
        await ctx.edit("❌ Cancelled")

    await ctx.close()

markup = EmojiKeyBoard(
    rows=[[
        EmojiButton(emoji="✅", data="yes"),
        EmojiButton(emoji="❌", data="no"),
    ]],
    callback=on_confirm,
)

await utils.answer(
    mx,
    "Confirm action:",
    event=event,
    reply_markup=markup,
)
```

### 2. Pagination (pages)

```python
PAGES = ["page 1", "page 2", "page 3"]

async def on_page(ctx):
    page = (ctx.data.get("page", 0) + 1) % len(PAGES)
    ctx.data["page"] = page
    await ctx.edit(PAGES[page])

markup = EmojiKeyBoard(
    rows=[[EmojiButton(emoji="➡️", data="next")]],
    callback=on_page,
    data={"page": 0},
    remove_clicked=False,
)

await utils.answer(
    mx,
    PAGES[0],
    event=event,
    reply_markup=markup,
)
```

### 3. Rating

```python
async def on_rate(ctx):
    await ctx.edit(f"⭐ Rating: {ctx.payload}/5")
    await ctx.close()

markup = EmojiKeyBoard(
    rows=[[
        EmojiButton(emoji="⭐", data=1),
        EmojiButton(emoji="⭐⭐", data=2),
        EmojiButton(emoji="⭐⭐⭐", data=3),
    ]],
    callback=on_rate,
)

await utils.answer(
    mx,
    "Rate:",
    event=event,
    reply_markup=markup,
)
```

### 4. Action Menu (dict in data)

```python
async def on_action(ctx):
    action = ctx.payload.get("action")

    if action == "refresh":
        ctx.data["count"] = ctx.data.get("count", 0) + 1
        await ctx.edit(
            f"Counter: {ctx.data['count']}",
        )
    elif action == "close":
        await ctx.close()

markup = EmojiKeyBoard(
    rows=[
        [EmojiButton(emoji="🔄", data={"action": "refresh"})],
        [EmojiButton(emoji="🧹", data={"action": "close"})],
    ],
    callback=on_action,
    data={"count": 0},
)
```

### 5. Editing with media

When using `utils.answer()` with media (`Image`, `Video`, etc.) and `reply_markup`, always pass `edit_id` explicitly so reactions are attached to the visible event:

```python
# ❌ Wrong — reactions go to the invisible edit event
await utils.answer(mx, media=Image(url=url), reply_markup=markup)

# ✅ Correct — use edit_id of the message being edited
await utils.answer(mx, media=Image(url=url), edit_id=message_id, reply_markup=markup)
```

This ensures `EmojiKeyBoard` reactions appear on the edited message, not on a hidden internal event.
```
