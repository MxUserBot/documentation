# WHY DOES THIS EXIST?

## Advantages

### 1. You are the bot

The userbot runs under your account. No need to create a separate bot, configure it, or invite it to rooms. All your rooms are available immediately.

### 2. Minimal external dependencies

All you need is Python + access to a Matrix server. No need for:
- A separate server for the bot
- Complex configuration

Basically: install UV, clone the repo, run it — it works.

### 3. Built-in module repository

The system repository is connected by default: `https://raw.githubusercontent.com/MxUserBot/mx-modules/main`

Any module from there can be installed with a single command: `.mdl module_name`
You can add your own repositories: `.addrepo https://github.com/user/repo`


### 4. Emoji callbacks instead of buttons

Reactions as buttons — works everywhere, why not? Since there are no inline buttons.

### 5. Web panel

FastAPI web UI for first login, module management, and config. No need to touch the console.

### 6. Community code security

Community modules run in a sandbox: AST analysis on load, audit hook on file/socket/import, ScopedDatabase, frozen core modules. A malicious module cannot leak keys or execute shell commands.

### 7. SAS verification

Device verification directly from the bot, without third-party tools. Full cycle: request → emoji → MAC → done.
