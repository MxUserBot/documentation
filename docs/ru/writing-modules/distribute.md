# Распространение модулей

Ты написал модуль. Он прекрасен. Он делает что-то полезное (или бесполезное, но забавное). И теперь ты хочешь поделиться этим с миром.

Есть три способа распространить свои модули.

## Способ 1: Репозиторий (РЕКОМЕНДОВАННЫЙ)

Это цивилизованный способ. Как pip, npm, но для MXUserBot.

### Как это работает

Ты заливаешь модули на GitHub/GitLab/любой git хост, генерируешь `index.json` и говоришь юзерам добавить твой репозиторий. Всё.

### Шаг за шагом

 1. **Создай структуру:**
    ```
    твой-репо/                    ← имя твоего git репозитория
    ├── modules/                  ← ПАПКА modules (всегда так называется)
    │   ├── wikipedia.py          ← одиночный файл-модуль
    │   ├── calculator.py         ← ещё один одиночный
    │   └── myepicmodule/         ← ПАПКА = ZIP-модуль [много файлов]
    │       ├── __init__.py
    │       └── utils.py
    └── index.json                ← сгенерируется скриптом ниже
    ```

2. **Поменяй `BASE_URL` в скрипте на свой:**
   
   Формат:
   ```
   https://raw.githubusercontent.com/ТВОЙ_ЮЗЕРНЕЙМ/ИМЯ_РЕПО/main/modules
   ```
   
   Пример: если твой репо `https://github.com/miku1337/my-modules`, то будет:
   ```
   BASE_URL = "https://raw.githubusercontent.com/miku1337/my-modules/main/modules"
   ```

3. **Запусти скрипт — он создаст index.json и зазипует папочные модули**

4. **Залей всё на GitHub**

5. **Скажи юзерам подключить:**
   ```
   .addrepo https://raw.githubusercontent.com/ТВОЙ_ЮЗЕРНЕЙМ/ИМЯ_РЕПО/main
   ```

### Скрипт для генерации index.json

??? note "Тыкни сюда чтобы увидеть код (спойлер)"

    ```python
    import ast
    import io
    import json
    import zipfile
    from pathlib import Path

     BASE_URL = "https://raw.githubusercontent.com/ТВОЙ_ЮЗЕРНЕЙМ/ИМЯ_РЕПО/main/modules"  # ← ЗАМЕНИ НА СВОЙ!

    SKIP_DIRS = {"__pycache__"}
    SKIP_EXTS = {".pyc", ".pyo"}
    SKIP_FILES = {"__pycache__"}


    def extract_value(node):
        try:
            return ast.literal_eval(node)
        except Exception:
            return None


    def extract_meta(source: str, default_url: str) -> dict | None:
        try:
            tree = ast.parse(source)
        except (SyntaxError, UnicodeDecodeError):
            return None

        for node in ast.walk(tree):
            if isinstance(node, ast.ClassDef) and node.name == "Meta":
                meta = {
                    "url": default_url,
                    "author": "Unknown",
                    "dependencies": [],
                    "tags": [],
                }
                mapping = {
                    "name": "name",
                    "description": "description",
                    "version": "version",
                    "dependencies": "dependencies",
                    "tags": "tags",
                    "author": "author",
                }
                for item in node.body:
                    if isinstance(item, ast.Assign):
                        for target in item.targets:
                            if isinstance(target, ast.Name) and target.id in mapping:
                                val = extract_value(item.value)
                                if val is not None:
                                    meta[mapping[target.id]] = val
                return meta
        return None


    def package_folder(folder: Path) -> bytes:
        buf = io.BytesIO()
        with zipfile.ZipFile(buf, "w", zipfile.ZIP_DEFLATED) as zf:
            for path in folder.rglob("*"):
                if path.is_dir():
                    continue
                rel = path.relative_to(folder.parent)
                if path.name in SKIP_FILES or path.suffix in SKIP_EXTS:
                    continue
                if any(part in SKIP_DIRS for part in path.relative_to(folder).parts):
                    continue
                zf.write(path, str(rel))
        return buf.getvalue()


    def generate_index():
        folder_name = BASE_URL.rstrip("/").split("/")[-1]
        modules_dir = Path(folder_name)

        index_data = {}

        if not modules_dir.exists() or not modules_dir.is_dir():
            print(f"Папка {modules_dir} не найдена")
            return

        for file in sorted(modules_dir.glob("*.py")):
            if file.name.startswith("_"):
                continue

            meta = extract_meta(file.read_text("utf-8"), f"{BASE_URL.rstrip('/')}/{file.name}")
            if meta:
                index_data[file.stem] = meta
                print(f"Indexed: {file.stem}")
            else:
                print(f"Skipped (no Meta): {file.name}")

        for folder in sorted(modules_dir.iterdir()):
            if not folder.is_dir() or folder.name.startswith("_") or folder.name in SKIP_DIRS:
                continue

            init_file = folder / "__init__.py"
            if not init_file.exists():
                print(f"Skipped folder (no __init__.py): {folder.name}/")
                continue

            zip_name = f"{folder.name}.zip"
            zip_path = modules_dir / zip_name

            source = init_file.read_text("utf-8")
            meta = extract_meta(source, f"{BASE_URL.rstrip('/')}/{zip_name}")
            if not meta:
                print(f"Skipped folder (no Meta): {folder.name}/")
                continue

            data = package_folder(folder)
            zip_path.write_bytes(data)
            index_data[folder.name] = meta
            print(f"Packed & indexed: {folder.name}/ → {zip_name}")

        output_file = Path("index.json")
        output_file.write_text(
            json.dumps(index_data, indent=2, ensure_ascii=False),
            encoding="utf-8",
        )
        print(f"\nindex.json written ({len(index_data)} entries)")


    if __name__ == "__main__":
        generate_index()
    ```

### Как юзеры будут ставить твои модули

После `.addrepo <url>` модули доступны как:

```
.mdl USER/module_name
```

Например:
```
.mdl miku/wikipedia
```

### Веб-интерфейс

Проще через веб: `modules` → `store` → поиск.

### Про "community" репо

По умолчанию твой репозиторий помечен как `community` — это недоверенный источник. Юзер увидит предупреждение при установке. Ничего личного, просто безопасность.

### Хочешь в официальный репо?

Если хочешь чтобы твой модуль попал в основной репозиторий MXUserBot:

1. Склонируй [mx-modules](https://github.com/MxUserBot/mx-modules)
2. Добавь модуль в `modules/`
3. Запусти `script.py` для обновления `index.json`
4. Сделай PR (Pull Request)

---

## Способ 2: Поделиться файлом

Для быстрой разработки или если ты ленивый.

### Как это работает:

1. Ты кидаешь в комнату файл `wikipedia.py`
2. Юзер отвечает на это сообщение командой `.mdl dev`
3. Бот выдаёт варнинг: "Непроверенный источник, ты уверен?"
4. Юзер подтверждает — модуль установлен

Идеально для тестирования или когда лень заморачиваться с репозиторием.

---

## Способ 3: Прямая ссылка

Как способ 2, но вместо файла — прямая raw-ссылка:

```
.mdl dev https://raw.githubusercontent.com/USER/repo/main/module.py
```

Юзер получит такой же варнинг про непроверенный источник.

---

## Краткий чеат-лист

| Способ | Когда использовать | Уровень доверия |
|--------|-------------------|-----------------|
| Репозиторий | Публикация для других | community (варнинг) |
| Файл | Разработка, тесты | dev (варнинг) |
| Ссылка | Быстрый шаринг | dev (варнинг) |

**Совет:** Используй репозиторий. Это чисто, цивилизованно и позволяет юзерам обновлять модули через `.mdl update`.
