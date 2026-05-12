# Prefix Module

Changes the prefix that starts all userbot commands. Default is `.`
Lets you change it to any allowed symbol.

## Commands

### `.set_prefix <char>`

**Access:** OWNER

Sets a new command prefix. The prefix is saved to the DB and restored after restart.

```
.set_prefix !
# → ✅ | Prefix successfully changed to: !
```

After this, all commands are called with `!`:
```
!ping
!help loader
```

## Config

| Key | Default | Description |
|-----|---------|-------------|
| `allowed_symbols` | `!"./\,;:@#$%^&*-_+=?\|~` | List of allowed symbols |

## Validation

- Strictly 1 character (prefixes longer than 1 char are rejected)
- Must be in the allowed symbols list (`allowed_symbols`)
- Saved via `_db.set("core", "prefix", [new_prefix])`

## Details

- Prefix is stored as a list in the DB (compatible with multi-prefix system)
- Applies instantly — no reload needed
- Only supports special characters (letters and digits are not allowed to avoid conflicts with normal text)
- Falls back to `.` if no prefix is set
