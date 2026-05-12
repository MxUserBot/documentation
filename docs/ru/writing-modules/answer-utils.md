# Ответы и утилиты

## utils.answer — как отвечать

Основная функция для отправки ответов.

### Импорты

```python
from mxc import utils
from mxc.types import Image, Video, Audio, Document, Sticker
```

### Простые примеры

```python
# Просто ответить на сообщение
await utils.answer(mx, "Привет!", event=event)

# С HTML тегами
await utils.answer(mx, "<b>жирный</b> <code>код</code> <i>курсив</i>", event=event)

# Отредактировать своё же сообщение
msg = await utils.answer(mx, "Загружаю...", event=event)
await utils.answer(mx, "✅ Готово!", edit_id=msg)

# Отправить в другую комнату
await utils.answer(mx, "Привет из другой комнаты!", room_id="!abc:server.org")
```

### С медиа (картинки, видео и т.д.)

```python
# Картинка по URL
await utils.answer(
    mx,
    "Вот картинка:",
    media=Image(url="https://example.com/img.jpg", mimetype="image/jpeg"),
    event=event,
)

# Картинка из байтов (из request)
data = await utils.request("https://example.com/img.png", return_type="bytes")
await utils.answer(
    mx,
    media=Image(data=data, mimetype="image/png"),
    event=event,
)

# Видео
await utils.answer(
    mx,
    "Видео:",
    media=Video(url="https://example.com/video.mp4", mimetype="video/mp4"),
    event=event,
)

# Аудио
await utils.answer(
    mx,
    media=Audio(url="https://example.com/audio.mp3", mimetype="audio/mpeg"),
    event=event,
)

# Документ/файл
await utils.answer(
    mx,
    "Файл:",
    media=Document(
        url="https://example.com/file.pdf",
        mimetype="application/pdf",
        filename="document.pdf",
    ),
    event=event,
)

# Стикер
await utils.answer(
    mx,
    media=Sticker(url="mxc://server/abcd", mimetype="image/png"),
    event=event,
)
```

### С эмодзи-клавиатурой

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
    "Подтверди:",
    event=event,
    reply_markup=markup,
)
```

### С кастомными эмодзи

```python
await utils.answer(
    mx,
    "Смотри %aam% и %thumbs%",
    event=event,
    emoji_map={
        "aam": "mxc://server/emoji1",
        "thumbs": "mxc://server/emoji2",
    },
)
```

---

## Другие полезные функции utils

```python
# HTTP запросы
data = await utils.request("https://api.example.com/data", return_type="json")
img = await utils.request("https://example.com/img.png", return_type="bytes")

# Аргументы команды
raw = await utils.get_args_raw(mx, event)    # весь текст после команды
args = await utils.get_args(mx, event)       # список слов

# Reply (ответ на сообщение)
reply_evt = await utils.get_reply_event(mx, event)
reply_text = await utils.get_reply_text(mx, event)

# Проверки
if await utils.is_dm(mx, room_id): ...       # это личная комната?
prefix = await utils.get_prefix(mx)           # текущий префикс

# Экранирование
safe = utils.escape_html(user_input)          # экранировать HTML теги
clean = utils.normalize_text(raw_text)        # нормализовать текст

# История сообщений
msgs = await utils.fetch_room_messages(mx, room_id, limit=100)

# Файлы (безопасные для песочницы)
path = await utils.safe_save(data, "file.tmp")
await utils.safe_remove("file.tmp")

# Отправить медиа напрямую (без utils.answer)
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

---

Полный список всех функций: [Справочник utils](utils-reference.md)
