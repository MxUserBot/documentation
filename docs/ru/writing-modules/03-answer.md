# Ответы и утилиты

## utils.answer — как отвечать

```python
# Просто ответить
await utils.answer(mx, "Привет!", event=event)

# С HTML
await utils.answer(mx, "<b>bold</b> <code>code</code>", event=event)

# Отредактировать своё же сообщение
status = await utils.answer(mx, "Загружаю...", event=event)
await utils.answer(mx, "Готово!", edit_id=status)

# В другую комнату
await utils.answer(mx, "Привет!", room_id="!room:server")

# С эмодзи-клавиатурой
await utils.answer(mx, "Выбери:", event=event, reply_markup=markup)

# С кастомными эмодзи (%shortcode%)
await utils.answer(mx, "Смотри %aam%", event=event,
    emoji_map={"aam": "mxc://server/...", "thumbs": "mxc://server/..."})

# С медиа
await utils.answer(mx, media=Image(url=data, mimetype="image/jpeg"), event=event)
```

---

## Другие utils

```python
# HTTP запросы
data = await utils.request("https://api.example.com/data", return_type="json")
img = await utils.request("https://example.com/img.png", return_type="bytes")

# Аргументы команды
raw = await utils.get_args_raw(mx, event)
args = await utils.get_args(mx, event)

# Reply
reply_evt = await utils.get_reply_event(mx, event)
reply_text = await utils.get_reply_text(mx, event)

# Проверки
if await utils.is_dm(mx, room_id): ...
prefix = await utils.get_prefix(mx)

# Экранирование
safe = utils.escape_html(user_input)
clean = utils.normalize_text(raw_text)

# История
msgs = await utils.fetch_room_messages(mx, room_id, limit=100)

# Файлы
path = await utils.safe_save(data, "file.tmp")
await utils.safe_remove("file.tmp")

# Медиа напрямую
await utils.send_image(mx, room_id, media=Image(...))
await utils.send_sticker(mx, room_id, media=Sticker(...))

# Rich presence
await utils.set_rpc_media(mx, artist="Radiohead", album="OK Computer", track="Karma Police")
await utils.clear_rpc(mx)

# Pin
await utils.pin(mx, room_id, event_id)
await utils.pin_room(mx, room_id)

# Скачать медиа из сообщения
data, filename, mime, size = await utils.download_message_media(mx, event)

# Инфо о платформе
info = utils.get_platform()
```

Полный список: `../8-utils-reference.md`
