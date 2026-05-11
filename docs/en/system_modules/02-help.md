# HelperModule — Help and Management Center

**File:** `modules/help.py`

Central help module. Provides a list of all active modules, detailed information about each module (commands, config, description), and on-the-fly module configuration management.

## Commands

### `.help [module]`

Without arguments — displays a list of all active modules, paginated by 5. Navigate via emoji buttons ⬅️ ➡️.

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

With a module name — detailed information:
- Name and description
- Config section (if any) with current values
- List of commands with prefix and description

```
.help pingpong
# → 📦 | PingPong
#    ℹ️ | Description: Simple ping-pong + dm checker
#
#    🛠 | Commands:
#     • .ping — Check bot latency
```

If the module is not found — displays an error message with search by name and by class name.

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

Changes a module configuration parameter. Searches for the module by `active_modules` key, then by `Meta.name`. Checks for `config.set()`.

```
.cfg LoaderModule repo_warn_ok true
.cfg TranslateMaster api_key abc123
```

On success: `✅ | Config Updated: <key> for <mod> set to <val>`
On error: `❌ | Config Update Failed: Invalid key or validation error for <key>.`
