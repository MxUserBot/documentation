Модуль для выполнения shell-команд на сервере прямо из чата Matrix. Предоставляет интерфейс для запуска произвольных команд в subprocess с таймаутом и обработкой больших выводов.

## Команды

### `.sh <command>`

**Доступ:** OWNER

Выполняет shell-команду и возвращает вывод.

```
.sh uname -a
# → ⚙️ | Executing command...
# → 📟 | Command: uname -a
#    📤 | Output:
#    Linux mxuserbot 6.8.0-arch1-1 #1 SMP PREEMPT_DYNAMIC x86_64 GNU/Linux

.sh df -h
# → 📟 | Command: df -h
#    📤 | Output:
#    Filesystem      Size  Used Avail Use% Mounted on
#    dev             7.8G     0  7.8G   0% /dev
#    ...
```

## Защита: подтверждение sudo

Если команда содержит `sudo` в любом месте, бот запрашивает подтверждение через эмодзи:

```
.sh sudo reboot
# → ⚠️ | Security Confirmation
#    You are about to execute:
#    sudo apt update
#
#    Do you want to proceed?
#    [✅] [❌]
```

При подтверждении — команда выполняется. При отказе — отменяется.

## Обработка больших выводов

Если вывод команды превышает **1000 строк**, он автоматически отправляется как текстовый файл:

```
.sh dmesg
# → ⚙️ | Executing command...
# → 📟 | Command: dmesg
#    📄 | Output (15234 lines) — sent as file:
#    (output.txt прикреплён)
```

## ShellExecutor

Внутренний класс, управляющий выполнением команд:

### Параметры

| Параметр | Значение | Описание |
|----------|----------|----------|
| `TIMEOUT` | 60.0 с | Максимальное время выполнения команды |
| `MAX_INLINE_LINES` | 1000 | Порог для отправки вывода файлом |

## Обработка ошибок

- **TimeoutError** — `⏱️ | Command execution timeout (60s)`
- **Exception** — `❌ | Error executing command: <traceback>`