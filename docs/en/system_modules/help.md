# HelperModule — Help & Management Center

**File:** `modules/help.py`

Central help module. Shows a list of all active modules, detailed info for each module (commands, config, description), and on-the-fly configuration management.

## Commands

### `.help [module]`

Without arguments — shows a list of all active modules, paginated in groups of 5. Navigation via emoji buttons ⬅️ ➡️.

```
.help
# → 💠 | MxUserBot
#    Helper Center
#
#    Available modules (Page 1/3):
#
#    ▫️ HelperModule
#    Helper Centre
#    ⬥ help, info, cfg
#
#    ▫️ LoaderModule
#    Module Manager
#    ⬥ mdl, msearch, addrepo, ...
#
#    [⬅️] [➡️]
```

With a module name — shows detailed info:
- Name and description
- Config section (if any) with current values
- Command list with prefix and description

```
.help pingpong
# → 📦 | PingPong
#    ℹ️ | Description: Simple ping-pong + dm checker
#
#    🛠 | Commands:
#     • .ping — Check bot latency
```

If the module is not found, shows an error message with search by name and class name.

### `.info`

**Access:** EVERYONE

Sends a system info card as an image with overlaid text:
- Userbot version
- Repository link

```
.info
# → (image with version and link)
```


### `.cfg <module> <key> <value>`

**Access:** EVERYONE

Changes a configuration parameter for a module. Looks up the module by `active_modules` key, then by `Meta.name`. Checks for `config.set()`.

```
.cfg LoaderModule repo_warn_ok true
.cfg TranslateMaster api_key abc123
```

On success: `✅ | Config Updated: <key> for <mod> set to <val>`
On error: `❌ | Config Update Failed: Invalid key or validation error for <key>.`
