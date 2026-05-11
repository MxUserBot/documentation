---

## Декораторы

### `@loader.command(name=None, aliases=[], security=SUDO)`

Регистрирует метод модуля как команду. Имя команды — имя функции (или `name`), aliases — альтернативные имена. Уровень доступа по умолчанию `SUDO`.

```python
@loader.command()
async def ping(self, mx, event):
    """Check bot latency"""
    ...

@loader.command(aliases=["df"], security=loader.OWNER)
async def disk_free(self, mx, event):
    ...
```

### `@loader.watcher(regex, security=EVERYONE)`

Регистрирует watcher — функция выполняется на КАЖДОЕ сообщение, если оно соответствует regex. Компилируется с `re.IGNORECASE`. Запускается **после** команд, если сообщение не было обработано FSM и не является командой.

```python
@loader.watcher(r"(?:спс|спасибо|thx|thanks)", security=loader.EVERYONE)
async def auto_thanks(self, mx, event, match):
    await event.react("❤️")
```

**Процесс:** все watcher'ы всех модулей проверяются в `message_cb`
Если сообщение подходит под FSM или команду — watcher'ы не запускаются.

Подробнее: [Watcher'ы](#watcher)

### `@loader.on(event_type)`

Регистрирует обработчик произвольного события Matrix. Тип события — любой `EventType`. Хендлеры вызываются в `_dispatch_event()`.

```python
@loader.on(EventType.ROOM_NAME)
async def on_room_rename(self, mx, event):
    await utils.answer(mx, f"Room renamed!", event)
```

### `@loader.state(State)`

Регистрирует обработчик FSM-состояния. Вызывается, когда пользователь в состоянии `State` отправляет сообщение (без префикса).

```python
@loader.state(AskStates.name)
async def ask_name(self, mx, event, ctx: FSMContext):
    name = event.content.body.strip()
    await ctx.update_data(name=name)
    await ctx.set_state(AskStates.age)
```

**Процесс:** в `message_cb` сначала проверяется `fsm.get_state()`. Если состояние активно — ищется хендлер с `@loader.state(<текущее_состояние>)`. Если найден — запускается с `FSMContext` как дополнительный аргумент. Если сообщение с префиксом — состояние сбрасывается (finish).

Подробнее: [FSM](#fsm-finite-state-machine)

### `@loader.cron("30m")` / `@loader.cron("*/5 * * * *")`

Регистрирует периодическую задачу. Интервал — строка с суффиксом (`60s`, `30m`, `2h`) или cron-выражение (`*/5 * * * *` — каждые 5 минут).

```python
@loader.cron("30m")
async def auto_status(self, mx):
    await mx.client.set_status("online")
```

Подробнее: [Cron](#cron)

---

## FSM (Finite State Machine)

**Файл:** `mxc/src/mxc/fsm.py`

Машина состояний для диалогов с пользователем. Позволяет модулю задавать вопросы и получать ответы последовательно.

### Структура

```python
class FSM:
    _states = {
        "!room:server:@user:server": {
            "state": "AskStates:name",
            "data": {"name": "Alice"},
            "_expires_at": 1234567890.0  # если ttl > 0
        }
    }
    _processed_events: set[str]
```

---

## Watcher'ы

Watcher'ы — это обработчики, которые реагируют на текст сообщения по regex. Запускаются **только** если сообщение не прошло через FSM и не является командой.

### Как работают

```python
@loader.watcher(r"hello", security=EVERYONE)
async def on_hello(self, mx, event, match):
    ...
```

- Все watcher'ы всех модулей проверяются подряд
- Если несколько модулей имеют watcher на один regex — запустятся все
- Работает асинхронно через `asyncio.create_task`
- `match` передаётся как keyword-аргумент (может быть `match.groupdict()`, строка из groups, или `match.group(0)`)

---

## Cron

Периодические задачи, запускающиеся с фиксированным интервалом. Не зависят от сообщений.

### Парсинг интервала

`_parse_cron(s)` в `core/loader.py:123`:

| Формат | Пример | Результат |
|--------|--------|-----------|
| `N s/m/h` | `30m` | 1800с |
| `*/N * * * *` | `*/5 * * * *` | 300с |
| `0 * * * *` | `0 * * * *` | 3600с |
| остальное | — | 60с (fallback) |

---

## Rate Limiter (AIMD)

Защита от 429 (M_LIMIT_EXCEEDED) от Matrix-сервера. Реализует AIMD (Additive Increase Multiplicative Decrease) — адаптивный алгоритм контроля скорости запросов.

### Включение

В `__main__.py` при создании `MXCClient`:
```python
self.client = MXCClient(
    ...,
    rate_limit_protect=True  # → mautrix_rate_limit_patch()
)
```

Глобальный патч перехватывает `HTTPAPI._send` в mautrix и оборачивает все не-GET запросы в rate limiter.

### GlobalRateLimiter

**Параметры:**
| Параметр | Старт | Минимум | Максимум |
|----------|-------|---------|----------|
| `min_interval` | 0.5с | 0.4с | 15.0с |

**Алгоритм:**

```
acquire():
  1. Если backoff активен — спит до его окончания
  2. Выдерживает min_interval между запросами
  3. Возвращает управление

report_success():
  1. Если нет 429 за последние 60с:
     - Счётчик ok_since_last_429++
     - Если >= 15 успехов: min_interval *= 0.95 (ускорение)
  2. Если есть 429 за 60с: сброс счётчика

report_error() (на 429):
  1. backoff_until = now + max(min_interval * 2, recent_429_count * 3.0, до 15с)
  2. min_interval *= 1.50 (замедление, макс 15с)
```

### Retry logic

При получении 429 выполняется exponential backoff с jitter:
```
wait = min(2.0^attempt + random(0, 1), 30.0)
```
Максимум 5 повторных попыток.

### Состояние (state_str)

В логах:
```
interval=0.500s 429=0.0% (0/142) backoff=0.0s ok_seq=15
interval=2.531s 429=3.1% (5/161) backoff=2.1s ok_seq=0
```

- `interval` — текущий интервал между запросами
- `429` — процент rate limit'ов за последние 60с
- `backoff` — оставшееся время backoff
- `ok_seq` — сколько успешных запросов подряд без 429

Если проще: обычному юзеру не нужно беспокоится о том, что на его сервере есть жесткие лимиты на реакции/удаление и пр - рейт лимиттер не допустит того, что бы юзер постоянно ловил 429е ошибки.
Конечно, из-за этого работа юзербота может слегка замедлиться т.к с рейт лимиттером задержка запросов вырастает на 400мс, но а как ещё?

---