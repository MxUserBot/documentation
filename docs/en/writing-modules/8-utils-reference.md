# UTILS — FULL REFERENCE

Import: `from mxc import utils`

---

## 📨 Sending messages

### `utils.answer(mx, text=None, media=None, html=True, room_id=None, event=None, edit_id="-1", reply_markup=None, emoji_map=None)`

Universal sender. Supports text, HTML, media, editing, keyboards, custom emoji.

```python
# Text
await utils.answer(
    mx,
    "Hello!",
    event=event,
)

# HTML
await utils.answer(
    mx,
    "<b>bold</b> <code>code</code>",
    event=event,
)

# Editing
status = await utils.answer(
    mx,
    "Loading...",
    event=event,
)
await utils.answer(
    mx,
    "Done!",
    edit_id=status,
)

# To another room
await utils.answer(
    mx,
    "Hello!",
    room_id="!room:server",
)

# With keyboard
await utils.answer(
    mx,
    "Choose:",
    event=event,
    reply_markup=markup,
)

# With custom emoji (%shortcode%)
await utils.answer(
    mx,
    "Check out %aam%",
    event=event,
    emoji_map={"aam": "mxc://server/...", "love": "mxc://server/..."},
)

# With media
await utils.answer(
    mx,
    media=Image(url=data, mimetype="image/jpeg"),
    event=event,
)
```

If `edit_id` is not specified and the event is from the bot, it edits the bot's own message.

### `utils.pin(mx, room_id, event_id, unpin=False)`
### `utils.pin_room(mx, room_id)`
### `utils.unpin_room(mx, room_id)`

Pin a message or room.

---

## 🌐 HTTP

### `utils.request(url, method="GET", return_type="json", **kwargs)`

Any `aiohttp` kwargs.

```python
data = await utils.request(
    "https://api.example.com/data",
    return_type="json",
)
html = await utils.request(
    "https://example.com",
    return_type="text",
)
img = await utils.request(
    "https://example.com/image.png",
    return_type="bytes",
)
result = await utils.request(
    "https://api.example.com/submit",
    method="POST",
    json={"key": "value"},
    return_type="json",
)
```

---

## 📝 Arguments

### `utils.get_args_raw(mx, event) -> str`

Everything after the command name.

```python
raw = await utils.get_args_raw(mx, event)
```

Do **not** rely on automatic reply fetching — manually use `get_reply_event` / `get_reply_text` if you need the message the user replied to.

### `utils.get_args(mx, event) -> list`

Same, but `shlex.split()`.

```python
args = await utils.get_args(mx, event)
```

### `utils.get_prefix(mx) -> str`

Current prefix (default `.`).

---

## 🔁 Events

### `utils.get_reply_event(mx, event) -> MessageEvent | None`

The event being replied to. Resolves edits.

```python
reply = await utils.get_reply_event(mx, event)
if reply:
    sender = reply.sender
    text = reply.content.body
```

### `utils.get_reply_text(mx, event) -> str | None | bool`

Text of the reply. Returns `False` if none.

```python
text = await utils.get_reply_text(mx, event)
```

### `utils.is_dm(mx, room_id) -> bool`

Check DM.

### `utils.should_ignore_event(mx, evt) -> bool`

Check whether to ignore.

### `utils.fetch_room_messages(mx, room_id, limit=100, from_token=None, direction="b") -> dict`

Room history.

```python
resp = await utils.fetch_room_messages(
    mx,
    room_id,
    limit=500,
    direction="b",
)
for evt in resp.get("chunk", []):
    print(evt.get("content", {}).get("body"))
```

### `utils.decrypt_event(mx, event) -> bool`

Decrypt. On failure requests keys.

---

## 🖼 Media

### `utils.send_image(mx, room_id, media, text, html, edit_id, **kwargs)`
### `utils.send_video`, `send_audio`, `send_document`, `send_sticker`

Usually simpler via `utils.answer(mx, media=...)`.

```python
from mxc.types import Image, Video, Audio, Document, Sticker

await utils.send_image(
    mx,
    room_id,
    media=Image(
        url=bytes_data,
        mimetype="image/png",
    ),
)
```

Automatic: encryption, thumbnail for images, dimension detection.

### `utils.download_message_media(mx, event_or_content) -> (bytes, filename, mimetype, size)`

Download media from a message.

```python
data, filename, mime, size = await utils.download_message_media(
    mx,
    event,
)
```

### `utils.encrypt(mx, room_id, file_bytes, mime_type, filename) -> (url, EncryptedFile)`

Encrypt and upload.

---

## 🎵 Rich Presence

### `utils.set_rpc_media(mx, artist, album, track, length, complete, cover_art, player, streaming_link)`

```python
await utils.set_rpc_media(
    mx,
    artist="Radiohead",
    album="OK Computer",
    track="Karma Police",
    length=240,
    cover_art="https://...",
)
```

### `utils.set_rpc_activity(mx, name, details, image)`

```python
await utils.set_rpc_activity(
    mx,
    name="reading documentation",
)
```

### `utils.clear_rpc(mx)`

---

## 🔧 Formatting

### `utils.escape_html(text) -> str`

`& < >` → entities.

### `utils.escape_quotes(text) -> str`

Same + quotes.

### `utils.normalize_text(text) -> str`

Remove HTML tags.

### `utils.get_platform() -> str`

OS, RAM, CPU in HTML.

### `utils.render_emojis(text, emoji_map) -> str`

Replaces `%shortcode%` with `<img>` tags using MXC links.

```python
from mxc.utils.emoji import render_emojis

html = render_emojis("Hello %aam%", {"aam": "mxc://..."})
# → 'Hello <img src="mxc://..." style="...">'
```

---

## 💾 Files

### `utils.safe_save(file_bytes, filename) -> str`

Save to `community/`.

```python
path = await utils.safe_save(data, "temp.jpg")
```

### `utils.safe_remove(filename)`

Delete from `community/`.

### `utils._get_safe_path(filename) -> Path`

Validate path.

### `utils.COMM_DIR -> Path`

Path to `community/`.

---

## 📦 Emoji Callbacks

```python
from mxc.types import EmojiButton
from mxc.utils.keyboard import EmojiKeyBoard
```

- `EmojiKeyBoard(rows, callback, ttl=0, ...)`
- `EmojiButton(emoji, data)`
- `EmojiCallbackContext` (payload, data, edit(), close(), react())

See `docs/3-emoji-callbacks.md`.

---

## 🛠 Misc

### `utils.get_commands(cls) -> dict`

Collect commands of a class.

### `utils.get_base_dir() -> str`, `get_dir(mod) -> str`

Project paths.

### `utils.convert_repo_url(url) -> str`

GitHub → raw URL.

```python
url = utils.convert_repo_url(
    "https://github.com/user/repo/tree/main/modules",
)
# → "https://raw.githubusercontent.com/user/repo/main/modules"
```

### `utils.starts_with_command(mx, body) -> bool`

Check prefix.
