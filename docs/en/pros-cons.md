# WHY EVEN BOTHER?

## Advantages

### 1. You are the bot

The userbot runs under your account. No need to create a separate bot, configure it, invite it to rooms. All your rooms are available right away.

### 2. Minimal external dependencies

All you need is Python + access to a Matrix server. No need for:
- A separate server for the bot
- Complex setup

Basically: install UV, clone the repo, run it — it works.

### 3. Built-in module repository

The system repository is connected by default.

Any module from there installs with one command:
```
.mdl module_name
```

You can add your own repositories:
```
.addrepo https://github.com/user/repo
```

### 4. Emoji callbacks instead of buttons

Matrix doesn't have inline buttons like Telegram. But it has reactions (emojis). And they work as buttons.

You send a message, the bot adds reactions. The user clicks a reaction — the bot does something. Works everywhere.

### 5. Rate Limiter — prevents you from getting banned

If you spam reactions, mass-delete messages — the Matrix server may give you a 429 (M_LIMIT_EXCEEDED) or even ban you.

Rate Limiter automatically:
- Slows down requests if the server complains
- Gradually speeds back up when things are fine
- Uses the AIMD algorithm like TCP

You don't need to worry.

### 6. Web panel

FastAPI web interface for first login, managing modules and config. No need to touch the console.

### 7. Community code security

Community modules run in a sandbox:
- AST analysis before loading
- Audit hook for all dangerous operations (file/socket/import)
- ScopedDatabase — each module only sees its own data
- Frozen core modules — built-in modules can't be modified

A malicious module can't leak your keys or execute shell commands.

### 8. SAS verification

Verify devices directly from the bot, without external tools. Full cycle:
```
request → emoji → MAC → done
```
