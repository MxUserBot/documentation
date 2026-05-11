## Как это работает?

Ты отправляешь сообщение, бот ставит реакции. Пользователь тыкает на реакцию → бот дёргает твой callback.

```
Сообщение: "Выбери язык:"
Реакции:  🐍 🦀 ☕

Пользователь тыкает 🦀
  → коллбек ловит реакцию
  → Вызывает callback с payload="Rust"
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
    "Текст сообщения",
    event=event,
    reply_markup=markup,
)
```

## Все параметры EmojiKeyBoard

| Параметр | Что делает | По умолчанию |
|----------|-----------|-------------|
| `rows` | Кнопки по рядам | **обязательно** |
| `callback` | Твоя функция | **обязательно** |
| `ttl` | Через сколько закроется (0 = бесконечно) | 0 |
| `allowed_senders` | Кто может тыкать | только отправитель |
| `remove_clicked` | Убирать реакцию после нажатия | True |
| `keep_reactions` | Восстанавливать если удалили | True |
| `allow_sudo` | Разрешить sudo тыкать | True |
| `data` | Словарь-состояние | {} |

## EmojiCallbackContext

Что получаешь в callback:

| Поле | Тип | Что содержит |
|------|-----|-------------|
| `ctx.payload` | Any | data из EmojiButton |
| `ctx.data` | dict | mutable словарь сессии |
| `ctx.key` | str | Эмодзи-ключ |
| `ctx.sender` | str | Кто нажал |
| `ctx.message_id` | str | ID сообщения |
| `ctx.room_id` | str | ID комнаты |

Методы:
- `await ctx.edit("текст")` — изменить сообщение
- `await ctx.react("👍")` — добавить реакцию
- `await ctx.close()` — закрыть + убрать реакции
- `await ctx.refresh()` — восстановить удалённые реакции

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