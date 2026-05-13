# ZIP модули

Когда модуль разрастается до 300+ строк, держать всё в одном файле становится больно. ZIP-модуль — это упакованная папка (package) с `__init__.py` и вспомогательными файлами.

```
my_module.zip
└── my_module/                  <- корневая папка ZIP (имя любое)
    ├── __init__.py             <- точка входа (обязательно)
    ├── helpers.py              <- вспомогательные функции
    ├── models.py               <- модели
    └── services/
        ├── __init__.py
        └── api.py              <- логика API
```

## Как это работает?

1. Бот распаковывает ZIP во временную папку
2. Ищет корневую директорию с `__init__.py`
3. AST-парсит `__init__.py` — проверяет `class Meta`
4. Копирует всю папку в `community/<name>/`
5. регистрирует модуль

## Как файлы связываются между собой

В `__init__.py` ты импортируешь классы и функции из других файлов обычным Python-способом:

```python
# community/my_module/__init__.py
from ..core import loader, utils

# Импорт из своих же файлов
from .helpers import format_result
from .models import MyPayload
from .services.api import fetch_data

class Meta:
    name = "MyZIPModule"
    description = "модуль из ZIP"
    version = "1.0.0"
    tags = ["example"]
    dependencies = ["aiohttp"]

@loader.tds
class MyZIPModule(loader.Module):
    @loader.command()
    async def cmd(self, mx, event, payload: MyPayload):  # MyPayload из models.py
        data = await fetch_data(payload.text)              # из services/api.py
        result = format_result(data)                       # из helpers.py
        await utils.answer(mx, result, event=event)
```

Остальные файлы:

```python
# community/my_module/models.py
from pydantic import BaseModel, Field, model_validator, ConfigDict

class MyPayload(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True)
    text: str = Field(min_length=1)

    @model_validator(mode='before')
    @classmethod
    def parse(cls, v):
        if isinstance(v, str):
            return {"text": v}
        return v
```

```python
# community/my_module/helpers.py
def format_result(data: dict) -> str:
    return f"<b>Результат:</b> <code>{data.get('value')}</code>"
```

```python
# community/my_module/services/api.py
from ..core import utils

async def fetch_data(query: str) -> dict:
    return await utils.request(
        f"https://api.example.com/search?q={query}",
        return_type="json",
    )
```
## Как собрать ZIP

```python
import zipfile
from pathlib import Path

def build_zip(source_dir: str, output: str):
    name = Path(source_dir).name
    with zipfile.ZipFile(output, "w", zipfile.ZIP_DEFLATED) as zf:
        for path in Path(source_dir).rglob("*"):
            if path.is_file():
                zf.write(path, arcname=f"{name}/{path.relative_to(source_dir)}")
```

Или просто архивируешь папку любым архиватором в ZIP
## Структура __init__.py

`__init__.py` — точка входа. Он должен содержать:
1. `class Meta` — на уровне модуля
2. `class XxxModule(loader.Module)` — класс с командой
3. Импорты из своих же файлов (если нужно)

```python
# __init__.py — минимальный
from ..core import loader
from .logic import do_stuff

class Meta:
    name = "Demo"
    description = "..."
    version = "1.0.0"
    tags = ["demo"]

@loader.tds
class DemoModule(loader.Module):
    @loader.command()
    async def cmd(self, mx, event):
        result = await do_stuff()
        ...
```
## Когда ZIP, а когда один файл?

| Ситуация | Формат |
|----------|--------|
| До 200 строк | Один `.py` |
| 350-400+ строк | zip |
Хотя никто конечно, вас не заставляет: вы можете написать 3к строк в одном .py :D 


## Важно

- `__init__.py` — точка входа. Он **должен** содержать `class Meta` и **или** `class XxxModule`, или импортировать модуль из другого файла который содержит класс Module
- Все вспомогательные `.py` файлы импортятся рекурсивно
- Если в папке есть другие папки — они тоже импортируются (как submodules)
- `__init__.py` НЕ обязательно должен содержать `class Module` — он может быть импортирован из другого файла пакета
