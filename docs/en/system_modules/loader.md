# Loader Module

Module management: install, search, delete, and reload. Interacts with the repository system to load modules from external sources.

## Commands

### `.mdl [dev] <id | url | reply>`

**Access:** OWNER

Install a module. Supports three modes:

**1. By ID from repository:**
```
.mdl ping
# → ⏳ | Processing ping...
# → ✅ | Module ping.py loaded successfully!
```

Or if it's a community repository: .mdl Miku/module

**2. By direct link:**
```
.mdl dev https://raw.githubusercontent.com/user/repo/main/module.py
```

**3. By replying to a file:**
Reply to a message with a .py or .zip file:
```
.mdl dev
# (reply to module.py)
```

**Security system:**

When installing from an untrusted source, the bot asks for confirmation:
```
⚠️ | SECURITY WARNING
You are installing a module from community repository —
an UNVERIFIED source.
This module has NOT been reviewed and may contain malicious code.

Do you confirm that you want to install this module?
[✅] [❌]
```

After confirming, the warning won't show again for that source type.

### `.msearch <query>`

**Access:** OWNER

Search for modules across all connected repositories.

```
.msearch ping
# → ✅ | Found in SYSTEM Repo: https://raw.githubusercontent.com/MxUserBot/mx-modules/main
#    📦 | PingPong (ping) v1.1.0
#    📥 | .mdl ping
```

Results are grouped by repository. If there are more than 3 results, pagination via ⬅️ ➡️ is enabled.

**Source markers:**
- ✅ SYSTEM — trusted modules from the main repository
- 👥 COMMUNITY — third-party modules (require `repo` prefix: `.mdl repo owner/module`)

### `.addrepo <url>`

**Access:** OWNER

Add a third-party module repository.

```
.addrepo https://raw.githubusercontent.com/user/custom-modules/main
```

Checks if index.json is available at the URL. If not — `❌ | Invalid repo or index!`. If it exists — asks for security confirmation.

### `.delrepo <url>`

**Access:** OWNER

Remove a repository from the list.

```
.delrepo https://raw.githubusercontent.com/user/custom-modules/main
# → ✅ | Repository removed.
```

### `.reload`

**Access:** OWNER

Full reload of all modules (rereads all files, re-registers commands).

```
.reload
# → ⏳ | Reloading all modules...
# → ♻️ | Modules reloaded. Total: 42
```

Calls `loader.register_all(mx)` — reinitializes all modules and updates `mx.active_modules`.

### `.unmd <name>`

**Access:** OWNER

Remove an installed module.

```
.unmd ping
# → ✅ | Module ping unloaded.
```
