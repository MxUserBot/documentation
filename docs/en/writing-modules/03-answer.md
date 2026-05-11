# Answer and Utilities

## utils.answer — how to reply

```python
# Simply reply
await utils.answer(mx, "Hello!", event=event)

# With HTML
await utils.answer(mx, "<b>bold</b> <code>code</code>", event=event)

# Edit your own message
status = await utils.answer(mx, "Loading...", event=event)
await utils.answer(mx, "Done!", edit_id=status)

# To a different room
await utils.answer(mx, "Hello!", room_id="!room:server")

# With emoji keyboard
await utils.answer(mx, "Choose:", event=event, reply_markup=markup)

# With custom emojis (%shortcode%)
await utils.answer(mx, "Check %aam%", event=event,
    emoji_map={"aam": "mxc://server/...", "thumbs": "mxc://server/..."})

# With media
await utils.answer(mx, media=Image(url=data, mimetype="image/jpeg"), event=event)
```

---

## Other utils

```python
# HTTP requests
data = await utils.request("https://api.example.com/data", return_type="json")
img = await utils.request("https://example.com/img.png", return_type="bytes")

# Command arguments
raw = await utils.get_args_raw(mx, event)
args = await utils.get_args(mx, event)

# Reply
reply_evt = await utils.get_reply_event(mx, event)
reply_text = await utils.get_reply_text(mx, event)

# Checks
if await utils.is_dm(mx, room_id): ...
prefix = await utils.get_prefix(mx)

# Escaping
safe = utils.escape_html(user_input)
clean = utils.normalize_text(raw_text)

# History
msgs = await utils.fetch_room_messages(mx, room_id, limit=100)

# Files
path = await utils.safe_save(data, "file.tmp")
await utils.safe_remove("file.tmp")

# Direct media
await utils.send_image(mx, room_id, media=Image(...))
await utils.send_sticker(mx, room_id, media=Sticker(...))

# Rich presence
await utils.set_rpc_media(mx, artist="Radiohead", album="OK Computer", track="Karma Police")
await utils.clear_rpc(mx)

# Pin
await utils.pin(mx, room_id, event_id)
await utils.pin_room(mx, room_id)

# Download media from message
data, filename, mime, size = await utils.download_message_media(mx, event)

# Platform info
info = utils.get_platform()
```

Full reference: `../8-utils-reference.md`
