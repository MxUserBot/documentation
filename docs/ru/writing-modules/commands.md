# Команды и парсинг аргументов

## Сигнатура команды

```python
async def cmd(self, mx, event):
```

**обязательно**:
- `self` — модуль
- `mx` — MXBotInterface (client, security, fsm, ...)
- `event` — MessageEvent

## Декоратор

```python
@loader.command()
@loader.command(name="hi")
@loader.command(aliases=["hello", "hi"])
@loader.command(security=loader.EVERYONE)
```

## 3 вида парсинга аргументов

### Тип 1: Простая типизация

Киньте в аргументы переменные, которые вам нужны.
аргумент: тип = что-то
```python
@loader.command()
async def tr(self, mx, event, lang: str = "en", text: str = None):
    """<lang> <text> - перевод"""
    # .tr ru привет мир → lang="ru", text="привет мир"
```
Если по умолчанию будет None, и функция ничего не подставит вместо None - вызовется Help этой функции.

На пример мы укажем .tr 
переменная text останется None.

В тоже время, мы можем брать текст из реплая на сообщение.

### Тип 2: Pydantic BaseModel

```python
from pydantic import BaseModel, Field, model_validator, ConfigDict

class SplitPayload(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True)
    top: str = Field(min_length=1)
    bottom: str = Field(default="")

    @model_validator(mode='before')
    @classmethod
    def parse(cls, v):
        if isinstance(v, str):
            parts = v.split("|", 1)
            return {"top": parts[0], "bottom": parts[1] if len(parts) > 1 else ""}
        return v


@loader.command()
async def cmd(self, mx, event, payload: SplitPayload):
    """<top> | <bottom> - демо"""
    # .cmd привет | мир → payload.top="привет", payload.bottom="мир"
```

**Что даёт:**
- Авто-валидация (min_length, ge, regex)
- Гибкий парсинг (split, regex, что угодно)

### Тип 3: Ручной парсинг

```python
@loader.command()
async def rp(self, mx, event):
    """<action> [@user] - RP команда"""
    raw = await utils.get_args_raw(mx, event)
    parts = raw.split(maxsplit=2)
    subcmd = parts[0].lower() if parts else ""
    # полная свобода
```

### Когда что?

смотрите по ситуации: что вы больше любите - то и юзайте. Не нравиться пудантик? Юзайте ручной парсинг, или в аргументах функции.

---

## config

```python
config = {
    "api_key": loader.ConfigValue(
        default=None, description="API ключ", required=True,
    ),
    "delay": loader.ConfigValue(
        default=5, description="Задержка (сек)",
        validator=lambda x: isinstance(x, int) and x >= 0,
    ),
    "secret": loader.ConfigValue(
        default="", description="Секрет", forbid=True,
    ),
}
```

- `required=True` — не даст юзать команды пока не заполнено. **Обязательно указывай `default=None`**, а не пустую строку.
- `forbid=True` — запретить изменять.
- `validator` — вернуть True/False

Доступ: `self.config.get("key")` / `self.config["key"]`
Установка: `self.config.set("key", value)` / `await self.config.set_async("key", value)`

---
