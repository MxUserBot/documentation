# UTILS — ПОЛНЫЙ СПРАВОЧНИК

Импорт: `from mxc import utils`

---

## 📨 Отправка сообщений

### `utils.answer(mx, text=None, media=None, html=True, room_id=None, event=None, edit_id="-1", reply_markup=None, emoji_map=None)`

Универсальная отправка. Умеет текст, HTML, медиа, редактирование, клавиатуры, кастомные эмодзи.

```python
# Текст
await utils.answer(
    mx,
    "Привет!",
    event=event,
)

# HTML
await utils.answer(
    mx,
    "<b>bold</b> <code>code</code>",
    event=event,
)

# Редактирование
status = await utils.answer(
    mx,
    "Загружаю...",
    event=event,
)
await utils.answer(
    mx,
    "Готово!",
    edit_id=status,
)

# В другую комнату
await utils.answer(
    mx,
    "Привет!",
    room_id="!room:server",
)

# С клавиатурой
await utils.answer(
    mx,
    "Выбери:",
    event=event,
    reply_markup=markup,
)

# С кастомными эмодзи (%shortcode%)
await utils.answer(
    mx,
    "Смотри %aam%",
    event=event,
    emoji_map={"aam": "mxc://server/...", "love": "mxc://server/..."},
)

# С медиа
await utils.answer(
    mx,
    media=Image(url=data, mimetype="image/jpeg"),
    event=event,
)
```

Если `edit_id` не указан, а event от бота — редактирует его же сообщение.

### `utils.pin(mx, room_id, event_id, unpin=False)`
### `utils.pin_room(mx, room_id)`
### `utils.unpin_room(mx, room_id)`

Закрепить сообщение или комнату.

---

## 🌐 HTTP

### `utils.request(url, method="GET", return_type="json", **kwargs)`

Любые `aiohttp` kwargs.

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

## 📝 Аргументы

### `utils.get_args_raw(mx, event) -> str`

Всё после имени команды.

```python
raw = await utils.get_args_raw(mx, event)
```

**Не полагайся** на авто-получение реплая — если нужен текст сообщения, на которое ответил пользователь, используй `get_reply_event` / `get_reply_text` вручную.

### `utils.get_args(mx, event) -> list`

То же, но `shlex.split()`.

```python
args = await utils.get_args(mx, event)
```

### `utils.get_prefix(mx) -> str`

Текущий префикс (по умолчанию `.`).

---

## 🔁 События

### `utils.get_reply_event(mx, event) -> MessageEvent | None`

Событие на которое отвечают. Расшифровывает, ищет редакции.

```python
reply = await utils.get_reply_event(mx, event)
if reply:
    sender = reply.sender
    text = reply.content.body
```

### `utils.get_reply_text(mx, event) -> str | None | bool`

Текст reply. Если нет — `False`.

```python
text = await utils.get_reply_text(mx, event)
```

### `utils.is_dm(mx, room_id) -> bool`

Проверить DM.

### `utils.should_ignore_event(mx, evt) -> bool`

Проверить надо ли игнорировать.

### `utils.fetch_room_messages(mx, room_id, limit=100, from_token=None, direction="b") -> dict`

История комнаты.

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

Расшифровать. При неудаче запрашивает ключи.

---

## 🖼 Медиа

### `utils.send_image(mx, room_id, media, text, html, edit_id, **kwargs)`
### `utils.send_video`, `send_audio`, `send_document`, `send_sticker`

Обычно проще через `utils.answer(mx, media=...)`.

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

Автоматически: шифрование, thumbnail для изображений, определение размеров.

### `utils.download_message_media(mx, event_or_content) -> (bytes, filename, mimetype, size)`

Скачать медиа из сообщения.

```python
data, filename, mime, size = await utils.download_message_media(
    mx,
    event,
)
```

### `utils.encrypt(mx, room_id, file_bytes, mime_type, filename) -> (url, EncryptedFile)`

Зашифровать и загрузить.

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
    name="читает документацию",
)
```

### `utils.clear_rpc(mx)`

---

## 🔧 Форматирование

### `utils.escape_html(text) -> str`

`& < >` → сущности.

### `utils.escape_quotes(text) -> str`

То же + кавычки.

### `utils.normalize_text(text) -> str`

Удалить HTML теги.

### `utils.get_platform() -> str`

ОС, RAM, CPU в HTML.

### `utils.render_emojis(text, emoji_map) -> str`

Заменяет `%shortcode%` на `<img>` теги с MXC-ссылками.

```python
from mxc.utils.emoji import render_emojis

html = render_emojis("Привет %aam%", {"aam": "mxc://..."})
# → 'Привет <img src="mxc://..." style="...">'
```

---

## 💾 Файлы

### `utils.safe_save(file_bytes, filename) -> str`

Сохранить в `community/`.

```python
path = await utils.safe_save(data, "temp.jpg")
```

### `utils.safe_remove(filename)`

Удалить из `community/`.

### `utils._get_safe_path(filename) -> Path`

Проверить путь.

### `utils.COMM_DIR -> Path`

Путь к `community/`.

---

## 📦 Эмодзи-коллбеки

```python
from mxc.types import EmojiButton
from mxc.utils.keyboard import EmojiKeyBoard
```

- `EmojiKeyBoard(rows, callback, ttl=0, ...)`
- `EmojiButton(emoji, data)`
- `EmojiCallbackContext` (payload, data, edit(), close(), react())

См. `docs/3-emoji-callbacks.md`.

---

## 🛠 Разное

### `utils.get_commands(cls) -> dict`

Собрать команды класса.

### `utils.get_base_dir() -> str`, `get_dir(mod) -> str`

Пути проекта.

### `utils.convert_repo_url(url) -> str`

GitHub → raw URL.

```python
url = utils.convert_repo_url(
    "https://github.com/user/repo/tree/main/modules",
)
# → "https://raw.githubusercontent.com/user/repo/main/modules"
```

### `utils.starts_with_command(mx, body) -> bool`

Проверить префикс.
