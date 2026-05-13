# FSM — машина состояний

**Что это?** FSM позволяет вести диалог с пользователем шаг за шагом.

**Когда использовать:**
- Нужно спросить несколько значений последовательно
- Форма регистрации, опроса, настройки
- Любая многошаговая логика

---

## StatesGroup и State

Сначала определяем состояния:

```python
from mxc.fsm import StatesGroup, State

class AskStates(StatesGroup):
    name = State()    # шаг 1: спросить имя
    age = State()     # шаг 2: спросить возраст
    city = State()    # шаг 3: спросить город
```

Каждый `State()` — это отдельный шаг диалога.

---

## FSMContext

В обработчик состояния передаётся `ctx` — контекст:

| Метод | Что делает |
|-------|-----------|
| `await ctx.update_data(key=val)` | Обновить данные сессии |
| `await ctx.get_data()` | Получить все данные как dict |
| `await ctx.set_state(NextState)` | Перейти к следующему шагу |
| `await ctx.clear()` | Сбросить состояние и данные |
| `await ctx.finish()` | Сбросить только состояние (данные сохранить) |

---

## Простейший пример БЕЗ Pydantic

```python
from mxc.fsm import StatesGroup, State
from mxc import utils
from .. import loader


class FormStates(StatesGroup):
    name = State()
    age = State()


@loader.tds
class FormModule(loader.Module):
    strings = {
        "ask_name": "Как тебя зовут?",
        "ask_age": "Приятно познакомиться, <b>{name}</b>! Сколько тебе лет?",
        "done": (
            "✅ Готово!<br><br>"
            "Имя: <code>{name}</code><br>"
            "Возраст: <code>{age}</code>"
        ),
    }

    @loader.command()
    async def form(self, mx, event):
        """Запустить опросник"""
        mx.fsm.set_state(event, FormStates.name)
        await utils.answer(mx, self.strings["ask_name"], event=event)

    @loader.state(FormStates.name)
    async def got_name(self, mx, event, ctx):
        """Пользователь ввёл имя"""
        text = event.content.body.strip()
        await ctx.update_data(name=text)
        await ctx.set_state(FormStates.age)
        await utils.answer(
            mx,
            self.strings["ask_age"].format(name=text),
            event=event,
        )

    @loader.state(FormStates.age)
    async def got_age(self, mx, event, ctx):
        """Пользователь ввёл возраст"""
        text = event.content.body.strip()
        await ctx.update_data(age=text)
        data = await ctx.get_data()
        await ctx.clear()
        await utils.answer(
            mx,
            self.strings["done"].format(**data),
            event=event,
        )
```

---

## Как это работает по шагам

| Шаг | Что происходит |
|-----|----------------|
| 1 | Пользователь пишет `.form` |
| 2 | Команда вызывает `mx.fsm.set_state(event, FormStates.name)` |
| 3 | Бот спрашивает: "Как тебя зовут?" |
| 4 | Пользователь отвечает: `Миша` — **ЭТО НЕ КОМАНДА, без точки** |
| 5 | Бот видит что есть активное состояние `FormStates.name` |
| 6 | Вызывает обработчик `@loader.state(FormStates.name)` |
| 7 | Обработчик берёт текст из сообщения через `event.content.body` |
| 8 | Сохраняет имя через `ctx.update_data()` |
| 9 | Переходит к следующему состоянию: `ctx.set_state(FormStates.age)` |
| 10 | И так далее... |
| 11 | В конце: `ctx.clear()` чтобы сбросить состояние |

---

## Проверка/валидация — вручную

Если нужно проверить что пользователь ввёл именно число:

```python
@loader.state(FormStates.age)
async def got_age(self, mx, event, ctx):
    text = event.content.body.strip()
    
    if not text.isdigit():
        await utils.answer(mx, "❌ Введи число!", event=event)
        return  # НЕ вызываем set_state — остаёмся в том же состоянии
    
    age = int(text)
    if age < 1 or age > 150:
        await utils.answer(mx, "❌ Нереальный возраст!", event=event)
        return
    
    await ctx.update_data(age=age)
```

**Если пользователь ошибся** — просто не вызывай `ctx.set_state()` и бот останется в том же состоянии.

---

## Команды прерывают FSM

Если пользователь в середине диалога напишет команду (с точкой):
```
.help
```

Состояние **автоматически сбросится**. Это фича — не баг.

---

## Больше примеров

### Пример 1: Да/Нет подтверждение

```python
class ConfirmStates(StatesGroup):
    confirm = State()


@loader.state(ConfirmStates.confirm)
async def got_confirm(self, mx, event, ctx):
    text = event.content.body.strip().lower()
    
    if text in ["да", "yes", "y", "д"]:
        await utils.answer(mx, "✅ Подтверждено!", event=event)
        await ctx.clear()
    elif text in ["нет", "no", "n", "н"]:
        await utils.answer(mx, "❌ Отменено", event=event)
        await ctx.clear()
    else:
        await utils.answer(mx, "Введи да или нет", event=event)
```

### Пример 2: Пропуск шага

```python
class OptionalStates(StatesGroup):
    name = State()
    bio = State()


@loader.state(OptionalStates.bio)
async def got_bio(self, mx, event, ctx):
    text = event.content.body.strip()
    
    if text == "-" or text == "skip":
        await ctx.update_data(bio="не указано")
    else:
        await ctx.update_data(bio=text)
```

### Пример 3: ctx.data — сохраняем промежуточные данные

```python
@loader.state(FormStates.name)
async def step1(self, mx, event, ctx):
    text = event.content.body.strip()
    await ctx.update_data(name=text, step=1)
    await ctx.set_state(FormStates.next)


@loader.state(FormStates.next)
async def step2(self, mx, event, ctx):
    data = await ctx.get_data()
    # data = {"name": "Миша", "step": 1}
```

---

## FSM — как это устроено

Состояния хранятся так:
```python
{
    "!room:server:@user:server": {
        "state": "FormStates:name",
        "data": {"name": "Миша"},
    }
}
```

Ключ = `room_id:user_id`. Один пользователь может быть только в одном состоянии одновременно.

---

## Pydantic — если хочется автоматическую валидацию

Если не хочется вручную проверять `isdigit()` и т.д., можно использовать Pydantic. Это опция.

```python
from pydantic import BaseModel, Field, model_validator, ConfigDict


class AgePayload(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True)
    age: int = Field(ge=1, le=150)

    @model_validator(mode='before')
    @classmethod
    def parse(cls, v):
        if isinstance(v, str):
            try:
                return {"age": int(v.strip())}
            except ValueError:
                raise ValueError("Введи число!")
        return v


@loader.state(FormStates.age)
async def got_age(self, mx, event, ctx, payload: AgePayload):
    await ctx.update_data(age=payload.age)
```

Если валидация не прошла — бот отправит ошибку и **останется в том же состоянии**.

---

**Для обычных людей:** Что такое FSM простыми словами — смотри [Ключевые концепции](../key-concepts.md#fsm-диалоги-шаг-за-шагом).
