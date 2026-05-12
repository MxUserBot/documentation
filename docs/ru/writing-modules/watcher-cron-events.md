# Watcher, Cron, Event Handler

## Watcher — наблюдатель за сообщениями

**Что это?** Watcher реагирует на ЛЮБОЕ сообщение по regex-шаблону, а не только на команды с префиксом.

**Важно:** Watcher'ы запускаются ТОЛЬКО если:
- Сообщение НЕ является командой (не начинается с `.`)
- Сообщение НЕ обрабатывается FSM

**Порядок обработки:**
1. Команды (`.ping`)
2. FSM (если активно)
3. Watcher'ы

---

## Сигнатура Watcher

```python
@loader.watcher(regex)
async def handler(self, mx, event, match):
```

- `regex` — строка с регулярным выражением
- `match` — то что нашлось по regex

---

## Примеры

### Авто-реакция на "спасибо"

```python
@loader.watcher(r"(?:^|\s)(?:спс|спасибо|thx|thanks)(?:$|\s)")
async def auto_thanks(self, mx, event, match):
    await event.react("❤️")
```

### Авто-конвертация валют

```python
import re

@loader.watcher(r"(\d+(?:\.\d+)?)\s*(\$|€|₽|rub|usd|eur)", flags=re.IGNORECASE)
async def auto_convert(self, mx, event, match):
    amount = match.group(1)
    currency = match.group(2).lower()
    # ... конвертация
```

---

## Про `match`

Что передаётся в `match`:
- Если regex имеет именованные группы `(?P<name>...)` → `match.groupdict()`
- Если regex имеет обычные группы → `match.groups()` или `match.group(1)`
- Если нет групп → `match.group(0)`

### С именованными группами

```python
@loader.watcher(r"перевести (?P<amount>\d+) (?P<currency>\w+)")
async def converter(self, mx, event, match: dict):
    # match = {"amount": "100", "currency": "usd"}
    amount = match["amount"]
    currency = match["currency"]
```

### Без групп

```python
@loader.watcher(r"привет")
async def on_hello(self, mx, event, match: str):
    # match = "привет"
    await utils.answer(mx, "И тебе привет!", event=event)
```

---

## Особенности Watcher'ов

- **Все watcher'ы запускаются** если regex совпадает. Если 5 модулей имеют watcher на одно и то же — запустятся все 5.
- **Асинхронно**: Запускаются через `asyncio.create_task()`
- **НЕ блокируют** другие watcher'ы и команды
- **Нет порядка**: Не гарантируется какой watcher запустится раньше

---

## Event Handler — любые события Matrix

Реагирует не только на сообщения, но на ЛЮБЫЕ события Matrix:

```python
from mautrix.types import EventType

@loader.on(EventType.ROOM_MEMBER)
async def on_member(self, mx, event):
    """Кто-то вошёл/вышел из комнаты"""
    if event.content.membership == "join":
        await utils.answer(mx, "Привет!", event=event)
```

**Популярные EventType:**
- `EventType.ROOM_MESSAGE` — новое сообщение
- `EventType.ROOM_MEMBER` — вступление/выход/приглашение
- `EventType.ROOM_NAME` — изменено название комнаты
- `EventType.ROOM_TOPIC` — изменён топик
- `EventType.REACTION` — добавлена реакция
- `EventType.ROOM_ENCRYPTED` — зашифрованное сообщение

---

## Cron — периодические задачи

Выполняется автоматически по расписанию. Не зависит от сообщений.

```python
@loader.cron("30m")
async def every_half_hour(self, mx):
    """Каждые 30 минут"""
    self.log.info("Прошло 30 минут!")
```

**Поддерживаемые форматы:**

| Формат | Пример | Интервал |
|--------|--------|----------|
| `N s` | `60s` | 60 секунд |
| `N m` | `30m` | 30 минут |
| `N h` | `2h` | 2 часа |
| Cron выражение | `*/5 * * * *` | каждые 5 минут |
| Cron выражение | `0 * * * *` | каждый час |

**Fallback:** Если формат непонятный — используется `60s`.

---

## Примеры Cron

### Каждые 5 минут (cron)

```python
@loader.cron("*/5 * * * *")
async def check_status(self, mx):
    """Проверять статус каждые 5 минут"""
    try:
        data = await utils.request("https://api.example.com/status")
        self.log.info(f"Status: {data}")
    except Exception as e:
        self.log.error(f"Check failed: {e}")
```

### Часовой бэкап

```python
@loader.cron("1h")
async def hourly_backup(self, mx):
    """Каждый час"""
    # ... сделать бэкап
```

---

## Особенности Cron

- **Запускается при старте**: Первое выполнение сразу при загрузке модуля
- **Останавливается при выгрузке**: Когда модуль удаляют — cron останавливается
- **Параметры**: Только `(self, mx)` — нет `event`
- **Не перекрываются**: Если задача выполняется дольше интервала — следующая запустится по расписанию

---

**Для обычных людей:** Простыми словами что это такое — смотри [Ключевые концепции](../key-concepts.md).
