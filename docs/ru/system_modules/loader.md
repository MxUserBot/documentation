Модуль управления модулями: установка, поиск, удаление и перезагрузка. Взаимодействует с системой репозиториев для загрузки модулей из внешних источников.

## Команды

### `.mdl [dev] <id | url | reply>`

**Доступ:** OWNER

Установка модуля. Поддерживает три режима:

**1. По ID из репозитория:**
```
.mdl ping
# → ⏳ | Processing ping...
# → ✅ | Module ping.py loaded successfully!
```

или если это камунити репозиторий: .mdl Miku/module

**2. По прямой ссылке:**
```
.mdl dev https://raw.githubusercontent.com/user/repo/main/module.py
```

**3. Ответом на файл:**
Ответьте на сообщение с .py или .zip файлом:
```
.mdl dev
# (reply to module.py)
```

**Система безопасности:**

При установке из непроверенного источника бот запрашивает подтверждение:
```
⚠️ | SECURITY WARNING
You are installing a module from community repository —
an UNVERIFIED source.
This module has NOT been reviewed and may contain malicious code.

Do you confirm that you want to install this module?
[✅] [❌]
```

После подтверждения предупреждение больше не показывается для данного типа источников 

### `.msearch <query>`

**Доступ:** OWNER

Поиск модулей во всех подключённых репозиториях.

```
.msearch ping
# → ✅ | Found in SYSTEM Repo: https://raw.githubusercontent.com/MxUserBot/mx-modules/main
#    📦 | PingPong (ping) v1.1.0
#    📥 | .mdl ping
```

Результаты группируются по репозиториям. Если результатов больше 3 — включается пагинация через эмодзи ⬅️ ➡️.

**Маркеры источников:**
- ✅ SYSTEM — проверенные модули из основного репозитория
- 👥 COMMUNITY — сторонние модули (требуют `repo` префикс: `.mdl repo owner/module`)

### `.addrepo <url>`

**Доступ:** OWNER

Добавление стороннего репозитория модулей.

```
.addrepo https://raw.githubusercontent.com/user/custom-modules/main
```

Проверяет, что по URL доступен index.json. Если нет — `❌ | Invalid repo or index!`. Если есть — запрашивает подтверждение безопасности.

### `.delrepo <url>`

**Доступ:** OWNER

Удаление репозитория из списка.

```
.delrepo https://raw.githubusercontent.com/user/custom-modules/main
# → ✅ | Repository removed.
```

### `.reload`

**Доступ:** OWNER

Полная перезагрузка всех модулей (перечитывает все файлы, перерегистрирует команды).

```
.reload
# → ⏳ | Reloading all modules...
# → ♻️ | Modules reloaded. Total: 42
```

Вызывает `loader.register_all(mx)` — повторно инициализирует все модули и обновляет `mx.active_modules`.

### `.unmd <name>`

**Доступ:** OWNER

Удаление установленного модуля.

```
.unmd ping
# → ✅ | Module ping unloaded.
```