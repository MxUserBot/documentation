# Эмодзи коллбеки

## Как это работает?

В Matrix нет инлайн-кнопок как в Telegram. Но есть реакции (эмодзи). И они работают как кнопки.

Ты отправляешь сообщение, бот ставит реакции. Пользователь тыкает на реакцию — бот дёргает твой callback.

**Пример:**
```
Сообщение: "Какой цвет волос у Мику Хацунэ?"
Реакции:  🔵 🟢 🟤

Пользователь тыкает 🔵
  → коллбек ловит реакцию
  → Вызывает callback с payload="blue"

Сообщение меняется на: "✅ Правильно! Сине-голубой!"
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
    "Подтверди действие:",
    event=event,
    reply_markup=markup,
)
```

---

## Все параметры EmojiKeyBoard

| Параметр | Что делает | По умолчанию |
|----------|-----------|-------------|
| `rows` | Кнопки по рядам | **обязательно** |
| `callback` | Твоя функция обработчик | **обязательно** |
| `ttl` | Через сколько закроется (0 = бесконечно) | 0 |
| `allowed_senders` | Кто может нажимать. По умолчанию: отправитель команды + sudo пользователи | Только отправитель + sudo |
| `remove_clicked` | Убирать реакцию после нажатия | True |
| `keep_reactions` | Восстанавливать если удалили | True |
| `data` | Словарь-состояние, передаётся в callback | `{}` |

---

## EmojiCallbackContext

Что получаешь в callback:

| Поле | Тип | Что содержит |
|------|-----|-------------|
| `ctx.payload` | Any | `data` из EmojiButton |
| `ctx.data` | dict | mutable словарь сессии из параметра `data` |
| `ctx.key` | str | Эмодзи который нажали |
| `ctx.sender` | str | Кто нажал (`@user:server`) |
| `ctx.message_id` | str | ID сообщения |
| `ctx.room_id` | str | ID комнаты |

**Методы:**
- `await ctx.edit("текст")` — изменить сообщение
- `await ctx.react("👍")` — добавить реакцию
- `await ctx.close()` — закрыть + убрать все реакции
- `await ctx.refresh()` — восстановить удалённые реакции

---

## ПАТТЕРНЫ

### 1. Подтверждение (Confirm)

```python
async def on_confirm(ctx):
    if ctx.payload == "yes":
        await ctx.edit("✅ Ок, сделано!")
    else:
        await ctx.edit("❌ Отменено")
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
    "Подтверди действие:",
    event=event,
    reply_markup=markup,
)
```

### 2. Пагинация (страницы)

```python
PAGES = ["стр 1", "стр 2", "стр 3"]

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

### 3. Рейтинг (оценка)

```python
async def on_rate(ctx):
    await ctx.edit(f"⭐ Оценка: {ctx.payload}/5")
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
    "Оцени:",
    event=event,
    reply_markup=markup,
)
```

### 4. Action Menu (словарь в data)

```python
async def on_action(ctx):
    action = ctx.payload.get("action")

    if action == "refresh":
        ctx.data["count"] = ctx.data.get("count", 0) + 1
        await ctx.edit(
            f"Счётчик: {ctx.data['count']}",
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

### 5. Редактирование в callback через ctx.edit()

Когда пользователь нажал на реакцию — используй `ctx.edit()` чтобы изменить сообщение:

```python
async def on_confirm(ctx):
    if ctx.payload == "yes":
        await ctx.edit("✅ Ты выбрал ДА")
    else:
        await ctx.edit("❌ Ты выбрал НЕТ")
    
    await ctx.close()
```

**ctx.edit() принимает те же параметры что и utils.answer:**
- `ctx.edit("текст")` — просто текст
- `ctx.edit("текст", emoji_map=...)` — с кастомными эмодзи
- НО: `ctx.edit()` НЕ принимает `reply_markup` (если нужно новую клавиатуру — используй utils.answer с edit_id)

---

### 6. Кастомные эмодзи через emoji_map

Можно использовать кастомные (серверные) эмодзи через shortcodes `%имя%`:

```python
# Отправить с кастомными эмодзи
await utils.answer(
    mx,
    "Погладил %aam%",
    event=event,
    emoji_map={
        "aam": "mxc://your-server.org/abc123",
    },
)

# Редактировать с кастомными эмодзи в callback
async def handler(ctx):
    await ctx.edit(
        "Ты нажал %custom_emoji%",
        emoji_map={
            "custom_emoji": "mxc://server.org/xyz",
        },
    )
```

**Формат:**
- В тексте пишешь `%shortcode%`
- В `emoji_map` передаёшь словарь: `{"shortcode": "mxc://..."}`

---

### 7. Редактирование с медиа

При использовании `utils.answer()` с медиа (`Image`, `Video`, ...) и `reply_markup` — обязательно передавай `edit_id` явно, чтобы реакции прикрепились к видимому событию:

```python
# ❌ Неправильно — реакции уйдут на невидимый edit-event
await utils.answer(mx, media=Image(url=url), reply_markup=markup)

# ✅ Правильно — передаём edit_id редактируемого сообщения
await utils.answer(mx, media=Image(url=url), edit_id=message_id, reply_markup=markup)
```

Иначе `EmojiKeyBoard` реакции окажутся на скрытом внутреннем событии и не будут видны на сообщении.

---

**Для обычных людей:** Что такое эмодзи-коллбеки простыми словами — смотри [Ключевые концепции](../key-concepts.md#эмодзи-коллбеки-реакции-как-кнопки).
