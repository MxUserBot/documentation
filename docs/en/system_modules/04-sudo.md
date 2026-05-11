# SecurityModule — Access Control

**File:** `src/mxuserbot/modules/sudo.py`
**Class:** `SecurityModule`
**Tags:** `settings`

## Description

Access rights management module. Allows granting SUDO permissions to other users, opening access to individual modules or commands, and granting temporary command access for a specified duration.

## Access Level System

- **OWNER** — bot owner (full access to everything)
- **SUDO** — trusted users (access to commands with `SUDO` level)
- **EVERYONE** — all users (public commands, e.g. `.help`)
- **mod_perms** — targeted access to modules/commands
- **tsec** — temporary command access by time

## Commands

### `.sudo add/rm/list @user:server`

**Access:** OWNER

Manage the SUDO users list.

```
.sudo add @friend:matrix.org
# → 👤 User @friend:matrix.org is now SUDO.

.sudo list
# → 👤 SUDO users:
#    • @friend:matrix.org

.sudo rm @friend:matrix.org
# → 👤 User @friend:matrix.org is no longer SUDO.
```

**Auto-extract MXID:** If you reply to a user's message, the bot automatically extracts the MXID from `formatted_body` (parses `href`). This allows adding to SUDO without manually typing the MXID.

**MXID Validation:** Checks against the regex `^@.+:.+$`.

### `.modaccess add/rm @user:server <name>`

**Access:** OWNER

Targeted access to a specific module (by class name) or command (by name).

```
.modaccess add @user:server PingPongModule   # entire PingPong module
.modaccess add @user:server ping              # only .ping command
.modaccess rm @user:server ping               # revoke access
```

Checks whether the specified module or command exists before applying.

### `.tsec @user:server <command> <minutes>`

**Access:** OWNER

Temporary access to a command for a specified number of minutes. After expiry the bot automatically sends a notification to the chat where access was granted.

```
.tsec @guest:server ping 5
# → ⏱ @guest:server now has 5 min. for command ping
```

After expiry:
```
⏰ | guest, your time is up. you can no longer use the ping command
```

## Data Structures

### mod_perms
```python
{
    "@user:server": ["PingPongModule", "ping", ...]
}
```
Stored in DB: `core.mod_perms`

### tsec_users
```python
[
    {
        "target": "@user:server",
        "command": "ping",
        "expires": 1234567890.0,
        "room_id": "!room:server"
    }
]
```
Stored in DB: `core.tsec_users`

## Details

- Access check happens in `core/security.py` via `check_access()`
- tsec lifetime is checked on every command invocation
- Expired tsec entries are automatically removed from the list
- All changes are saved to DB immediately
- Works in conjunction with `core/security.py` and `core/callback.py`
