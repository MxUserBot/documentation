Module for changing the prefix that starts all userbot commands. The default is `.`. This module allows changing it to any permitted symbol.

## Commands

### `.set_prefix <char>`

**Access:** OWNER

Sets a new command prefix. The prefix is saved in the database and restored after restart.

```
.set_prefix !
# → ✅ | Prefix successfully changed to: !
```

After this all commands are invoked with `!`:
```
!ping
!help loader
```

## Configuration

| Key | Default | Description |
|-----|---------|-------------|
| `allowed_symbols` | `!"./\,;:@#$%^&*-_+=?\|~` | List of allowed symbols |

## Validation

- Strictly 1 character (commands longer than 1 character are rejected)
- Must be in the allowed symbols list (`allowed_symbols`)
- Saved via `_db.set("core", "prefix", [new_prefix])`

## Details

- Prefix is stored as a list in the database (compatible with multi-prefix system)
- Instant application — no reload needed
- Only supports special characters (letters and digits are not allowed to avoid conflicts with regular text)
- Falls back to `.` if no prefix is set
