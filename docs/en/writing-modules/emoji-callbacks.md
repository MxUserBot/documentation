# Emoji Callbacks

## How Does It Work?

Matrix doesn't have inline buttons like Telegram. But it has reactions (emojis). And they work as buttons.

You send a message, the bot adds reactions. The user clicks a reaction — the bot fires your callback.

**Example:**
```
Message: "What color is Hatsune Miku's hair?"
Reactions:  🔵 🟢 🟤

User clicks 🔵
  → callback catches the reaction
  → Calls callback with payload="blue"

Message changes to: "✅ Correct! Teal-blue!"
```

---

## EmojiKeyBoard

```python
from mxc.types import EmojiButton
from mxc.utils.keyboard import EmojiKeyBoard

markup = EmojiKeyBoard(
    rows=[
        [
            EmojiButton(emoji="✅", data="yes"),
            EmojiButton(emoji="❌", data="no"),
        ],
    ],
    callback=my_handler,
)

await utils.answer(
    mx,
    "Confirm action:",
    event=event,
    reply_markup=markup,
)
```

---

## All EmojiKeyBoard Parameters

| Parameter | What it does | Default |
|-----------|-------------|---------|
| `rows` | Buttons arranged in rows | **required** |
| `callback` | Your handler function | **required** |
| `ttl` | Time to live in seconds (0 = forever) | 0 |
| `allowed_senders` | Who can click. Default: command sender + sudo users | Sender only + sudo |
| `remove_clicked` | Remove reaction after click | True |
| `keep_reactions` | Restore if deleted | True |
| `data` | State dict, passed to callback | `{}` |

---

## EmojiCallbackContext

What you get in the callback:

| Field | Type | What it contains |
|-------|------|------------------|
| `ctx.payload` | Any | `data` from EmojiButton |
| `ctx.data` | dict | mutable session dict from the `data` parameter |
| `ctx.key` | str | The emoji that was clicked |
| `ctx.sender` | str | Who clicked (`@user:server`) |
| `ctx.message_id` | str | Message ID |
| `ctx.room_id` | str | Room ID |

**Methods:**
- `await ctx.edit("text")` — edit the message
- `await ctx.react("👍")` — add a reaction
- `await ctx.close()` — close + remove all reactions
- `await ctx.refresh()` — restore deleted reactions

---

## PATTERNS

### 1. Confirm

```python
async def on_confirm(ctx):
    if ctx.payload == "yes":
        await ctx.edit("✅ Ok, done!")
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

### 2. Pagination

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

### 5. Editing in callback via ctx.edit()

When the user clicks a reaction — use `ctx.edit()` to change the message:

```python
async def on_confirm(ctx):
    if ctx.payload == "yes":
        await ctx.edit("✅ You chose YES")
    else:
        await ctx.edit("❌ You chose NO")
    
    await ctx.close()
```

**ctx.edit() accepts the same parameters as utils.answer:**
- `ctx.edit("text")` — plain text
- `ctx.edit("text", emoji_map=...)` — with custom emoji
- BUT: `ctx.edit()` does NOT accept `reply_markup` (if you need a new keyboard — use utils.answer with edit_id)

---

### 6. Custom Emoji via emoji_map

You can use custom (server) emoji via shortcodes `%name%`:

```python
# Send with custom emoji
await utils.answer(
    mx,
    "Petted %aam%",
    event=event,
    emoji_map={
        "aam": "mxc://your-server.org/abc123",
    },
)

# Edit with custom emoji in callback
async def handler(ctx):
    await ctx.edit(
        "You clicked %custom_emoji%",
        emoji_map={
            "custom_emoji": "mxc://server.org/xyz",
        },
    )
```

**Format:**
- In text, write `%shortcode%`
- In `emoji_map`, pass a dict: `{"shortcode": "mxc://..."}`

---

### 7. Editing with Media

When using `utils.answer()` with media (`Image`, `Video`, ...) and `reply_markup` — always pass `edit_id` explicitly so that reactions attach to the visible event:

```python
# ❌ Wrong — reactions will go to an invisible edit-event
await utils.answer(mx, media=Image(url=url), reply_markup=markup)

# ✅ Correct — pass edit_id of the message being edited
await utils.answer(mx, media=Image(url=url), edit_id=message_id, reply_markup=markup)
```

Otherwise, `EmojiKeyBoard` reactions will end up on a hidden internal event and won't be visible on the message.

---

**For regular folks:** What emoji callbacks are in simple terms — see [Key Concepts](../key-concepts.md#emoji-callbacks-reactions-as-buttons).
