# SECURITY: ACCESS CONTROL SYSTEM

## Access Levels

```python
OWNER     = 1  # bit 0
SUDO      = 2  # bit 1
EVERYONE  = 4  # bit 2
```

By default, all executable commands can be run by — **SUDO**.

### How to set?

```python
@loader.command()                             # SUDO
@loader.command(security=loader.OWNER)        # OWNER only
@loader.command(security=loader.EVERYONE)     # everyone
```

### OWNER
- The bot itself (whoami) — automatically
- Added via: `.sudo add @user:server`
- **Can do anything.** Passes all checks.

### SUDOS
- `.sudo add @user:server`
- Can run SUDO + EVERYONE commands
- **CANNOT** run OWNER commands

### EVERYONE
- Anyone in the room
- For public commands

### ModAccess
```python
modaccess add @user:module_name
```
Grants access to a specific module/command without sudo.

### TSEC (temporary)
```python
tsec @user:server command 300
```
Expires after N seconds.

## check_access — check chain

```
1. User in owners?     → ALLOW
2. User in sudos?      → ALLOW
3. Command EVERYONE?   → ALLOW
4. Command SUDO+sudo?  → ALLOW
5. Has mod_perms?      → check
6. Command in mod_perms?  → ALLOW
7. Class in mod_perms?    → ALLOW
8. Has tsec?              → ALLOW
9. → DENY
```

## Firewall (audit hook)

The bot hooks `sys.addaudithook()` on startup:

### AST code analysis
Before executing community module code, it's checked:
- **Forbidden attributes:** `crypto`, `crypto_enabled`, `device_id`
- **Forbidden imports:** `sys`, `subprocess`, `ctypes`, `importlib`, `shutil`, `socket`, `pty`, `builtins`
- **Forbidden functions:** `eval()`, `exec()`, `__import__()`

### Runtime blocking

| Operation | Block |
|-----------|-------|
| `open()` with `w/a/x/+` | Writing from community code |
| `open()` on core files | Access to `/core/` |
| `os.remove()` | Deleting core files |
| `os.rename()` | Renaming core files |
| `import` core | Importing core from community |
| `ctypes.*` | Memory access |

### Frozen core modules

```python
# Community code cannot modify a core module:
object.__setattr__(core_module, "attr", "val")
# → PermissionError: Core modules are frozen!
```

## ScopedDatabase

Community modules get `ScopedDatabase`:
- All keys are prefixed with `module_name:`
- Cannot read/write other modules' data

## How to make a command OWNER-only?

```python
# Via security
@loader.command(security=loader.OWNER)
async def admin_cmd(self, mx, event):
    ...

```

## Custom access system

```python
config = {
    "access": loader.ConfigValue(
        default="all",
        description="all, sudo, owner, or comma-separated mxid list",
    ),
}

def _has_access(self, mx, sender):
    conf = self.config.get("access", "all").strip().lower()

    if conf == "all":
        return True
    if conf == "owner":
        return mx.security.is_owner(sender)
    if conf == "sudo":
        return (
            sender in mx.security.sudos
            or mx.security.is_owner(sender)
        )

    allowed = [
        x.strip()
        for x in conf.replace("\n", ",").split(",")
        if x.strip()
    ]
    return sender in allowed
```

## Gate — protecting event handlers

```python
self.client.add_event_handler(
    EventType.ROOM_MEMBER,
    self.security.gate(cb.invite_cb),
)
```

If the sender is not OWNER/SUDO — the event is ignored.


Keep in mind that the security here isn't perfect, it won't protect against everything, so full responsibility lies with you and your users.
