# ZIP Modules

When a module grows past 300+ lines, keeping everything in one file becomes painful. A ZIP module is a packaged folder (package) with `__init__.py` and helper files.

```
my_module.zip
└── my_module/                  <- root folder (name can be anything)
    ├── __init__.py             <- entry point (required)
    ├── helpers.py              <- helper functions
    ├── models.py               <- models
    └── services/
        ├── __init__.py
        └── api.py              <- API logic
```

## How Does It Work?

1. The bot extracts the ZIP to a temp folder
2. Looks for the root directory with `__init__.py`
3. AST-parses `__init__.py` — checks for `class Meta`
4. Copies the entire folder to `community/<name>/`
5. Registers the module

## How Files Reference Each Other

In `__init__.py` you import classes and functions from other files the usual Python way:

```python
# community/my_module/__init__.py
from ..core import loader, utils

# Import from your own files
from .helpers import format_result
from .models import MyPayload
from .services.api import fetch_data

class Meta:
    name = "MyZIPModule"
    description = "module from ZIP"
    version = "1.0.0"
    tags = ["example"]
    dependencies = ["aiohttp"]

@loader.tds
class MyZIPModule(loader.Module):
    @loader.command()
    async def cmd(self, mx, event, payload: MyPayload):  # MyPayload from models.py
        data = await fetch_data(payload.text)              # from services/api.py
        result = format_result(data)                       # from helpers.py
        await utils.answer(mx, result, event=event)
```

Other files:

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
    return f"<b>Result:</b> <code>{data.get('value')}</code>"
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
## How to Build a ZIP

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

Or just archive the folder with any archiver as ZIP
## __init__.py Structure

`__init__.py` is the entry point. It must contain:
1. `class Meta` — at module level
2. `class XxxModule(loader.Module)` — class with commands
3. Imports from your own files (if needed)

```python
# __init__.py — minimal
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
## When ZIP vs When Single File?

| Situation | Format |
|-----------|--------|
| Up to 200 lines | Single `.py` |
| 350-400+ lines | zip |
Though of course, nobody's forcing you: you can write 3k lines in a single .py :D


## Important

- `__init__.py` is the entry point. It **must** contain `class Meta` and **either** `class XxxModule`, or import the Module from another file in the package
- All helper `.py` files are imported recursively
- If there are subfolders — they also get imported (as submodules)
- `__init__.py` does NOT have to contain `class Module` — it can be imported from another file in the package
