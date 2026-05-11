Module for executing shell commands on the server directly from a Matrix chat. Provides an interface for running arbitrary commands in a subprocess with timeout and large output handling.

## Commands

### `.sh <command>`

**Access:** OWNER

Executes a shell command and returns the output.

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

## Safety: sudo confirmation

If the command contains `sudo` anywhere, the bot asks for confirmation via emoji:

```
.sh sudo reboot
# → ⚠️ | Security Confirmation
#    You are about to execute:
#    sudo apt update
#
#    Do you want to proceed?
#    [✅] [❌]
```

On confirmation — the command executes. On denial — it is cancelled.

## Large output handling

If the command output exceeds **1000 lines**, it is automatically sent as a text file:

```
.sh dmesg
# → ⚙️ | Executing command...
# → 📟 | Command: dmesg
#    📄 | Output (15234 lines) — sent as file:
#    (output.txt attached)
```

## ShellExecutor

Internal class managing command execution:

### Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| `TIMEOUT` | 60.0 s | Maximum command execution time |
| `MAX_INLINE_LINES` | 1000 | Threshold for sending output as file |

## Error handling

- **TimeoutError** — `⏱️ | Command execution timeout (60s)`
- **Exception** — `❌ | Error executing command: <traceback>`
