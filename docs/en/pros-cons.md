# WHY EVEN BOTHER?

## Advantages

### 1. Commands in your name

The userbot runs under your account, commands are executed on your behalf. No need to use other bots (in most cases) — there are modules to cover your needs.

### 2. No separate homeserver needed

With the [rate limiter](key-concepts.md#rate-limiter--ban-protection) — the bot can run on any server, within the server's limits of course.

Main dependency: [uv](https://docs.astral.sh/uv/).

No complex setup required: everything is configured either through the web panel or via Matrix chat.

### 3. Built-in module repository

The system repository is connected by default.

Any module from there installs with one command:
```
.mdl miku
```

You can add third-party repositories, then modules install like:
```
.mdl Pasha/miku
```

Add a repository:
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

You don't need to worry.

### 6. Web panel

FastAPI web interface for first login, managing modules and config. Convenient UX if you don't want to set everything up through chat.

### 7. Community code security

Community modules run in a sandbox:
- AST analysis before loading
- Audit hook for all dangerous operations (file/socket/import)
- ScopedDatabase — each module only sees its own data
- Frozen core modules — built-in modules can't be modified

A malicious module can't leak your keys or execute shell commands.

### 8. SAS verification

Verification works in two ways:

1. **Bot verification** — so the bot can work properly in encrypted rooms and see past messages, you should verify it. Only SAS (emoji verification) is available. Wait 5 to 20 seconds — the bot accepts your request and you can confirm. Currently comparing your emojis with the bot's is not available. Will be added later. After verification, the bot will store both your keys and its own.

2. **Verifying another device** — works like this:
   - `.devices` — list all your devices
   - Find the device ID
   - `.verif <id>` — start verification
   - Bot sends a verification request to the chat — you confirm
   - Bot shows emojis — you compare, confirm

Done! The bot shares keys and marks the device as verified in the local database.

Details: [SAS Verification](writing-modules/sas-verification.md)
