# Replies and Utilities

## utils.answer — how to reply

Main function for sending responses.

### Imports

```python
from mxc import utils
from mxc.types import Image, Video, Audio, Document, Sticker
```

### Simple Examples

```python
# Just reply to a message
await utils.answer(mx, "Hello!", event=event)

# With HTML tags
await utils.answer(mx, "<b>bold</b> <code>code</code> <i>italic</i>", event=event)

# Edit your own message
msg = await utils.answer(mx, "Loading...", event=event)
await utils.answer(mx, "✅ Done!", edit_id=msg)

# Send to a different room
await utils.answer(mx, "Hello from another room!", room_id="!abc:server.org")
```

### With Media (images, video, etc.)

```python
# Image by URL
await utils.answer(
    mx,
    "Here's a picture:",
    media=Image(url="https://example.com/img.jpg", mimetype="image/jpeg"),
    event=event,
)

# Image from bytes (from request)
data = await utils.request("https://example.com/img.png", return_type="bytes")
await utils.answer(
    mx,
    media=Image(data=data, mimetype="image/png"),
    event=event,
)

# Video
await utils.answer(
    mx,
    "Video:",
    media=Video(url="https://example.com/video.mp4", mimetype="video/mp4"),
    event=event,
)

# Audio
await utils.answer(
    mx,
    media=Audio(url="https://example.com/audio.mp3", mimetype="audio/mpeg"),
    event=event,
)

# Document/file
await utils.answer(
    mx,
    "File:",
    media=Document(
        url="https://example.com/file.pdf",
        mimetype="application/pdf",
        filename="document.pdf",
    ),
    event=event,
)

# Sticker
await utils.answer(
    mx,
    media=Sticker(url="mxc://server/abcd", mimetype="image/png"),
    event=event,
)
```

### With Emoji Keyboard

```python
from mxc.types import EmojiButton
from mxc.utils.keyboard import EmojiKeyBoard

markup = EmojiKeyBoard(
    rows=[
        [EmojiButton(emoji="✅", data="yes"), EmojiButton(emoji="❌", data="no")],
    ],
    callback=my_handler,
)

await utils.answer(
    mx,
    "Confirm:",
    event=event,
    reply_markup=markup,
)
```

### With Custom Emoji

```python
await utils.answer(
    mx,
    "Check %aam% and %thumbs%",
    event=event,
    emoji_map={
        "aam": "mxc://server/emoji1",
        "thumbs": "mxc://server/emoji2",
    },
)
```

---

## Other Useful utils Functions

```python
# HTTP requests
data = await utils.request("https://api.example.com/data", return_type="json")
img = await utils.request("https://example.com/img.png", return_type="bytes")

# Command arguments
raw = await utils.get_args_raw(mx, event)    # full text after the command
args = await utils.get_args(mx, event)       # list of words

# Reply (response to a message)
reply_evt = await utils.get_reply_event(mx, event)
reply_text = await utils.get_reply_text(mx, event)

# Checks
if await utils.is_dm(mx, room_id): ...       # is this a DM?
prefix = await utils.get_prefix(mx)           # current prefix

# Escaping
safe = utils.escape_html(user_input)          # escape HTML tags
clean = utils.normalize_text(raw_text)        # normalize text

# Message history
msgs = await utils.fetch_room_messages(mx, room_id, limit=100)

# Files (sandbox-safe)
path = await utils.safe_save(data, "file.tmp")
await utils.safe_remove("file.tmp")

# Send media directly (without utils.answer)
await utils.send_image(mx, room_id, media=Image(...))
await utils.send_sticker(mx, room_id, media=Sticker(...))

# Rich presence
await utils.set_rpc_media(mx, artist="Radiohead", album="OK Computer", track="Karma Police")
await utils.clear_rpc(mx)

# Pin
await utils.pin(mx, room_id, event_id)
await utils.pin_room(mx, room_id)

# Download media from a message
data, filename, mime, size = await utils.download_message_media(mx, event)

# Platform info
info = utils.get_platform()
```

---

Full list of all functions: [Utils Reference](utils-reference.md)
