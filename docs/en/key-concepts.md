# Key Concepts

Here's what MXUserBot is made of, explained in simple terms. No code, just the big picture.

---

## Commands

**What's this?** Stuff you type with a dot. For example:
```
.ping    → bot replies "Pong!"
.help    → shows command list
.mdl     → manages modules
```

**How does it work?** The bot watches every message. If it starts with `.` — it looks up the matching command.

---

## Watcher

**What's this?** Commands that react to ANY message, not just ones starting with a dot.

**Example:**
- You type in chat: "thank you!"
- The bot automatically adds a ❤️ reaction
- Or automatically converts currency if it sees `$100` in a message

**Where is it used?** Auto-reactions, auto-conversion, spam filtering — anything that should run on every message.

---

## FSM — step-by-step dialogs

**What's this?** A state machine. Lets the bot have a conversation with the user, step by step.

**Example:**
```
You:   .feedback
Bot:   "What's your name?"
You:   "Mike"
Bot:   "Nice to meet you, Mike! How old are you?"
You:   "25"
Bot:   "Thanks! Your feedback has been saved."
```

**Why is it needed?** Without FSM you'd have to cram everything into one command:
```
.feedback Mike 25 "Everything's great!"
```
With FSM — the bot asks each question in turn.

---

## Cron — scheduled tasks

**What's this?** Something that runs automatically at set time intervals.

**Examples:**
- Every 30 minutes: check server status
- Every hour: send statistics
- Once a day: database backup

**Formats:**
- `30m` — every 30 minutes
- `2h` — every 2 hours
- `*/5 * * * *` — every 5 minutes (cron expression)

---

## Emoji Callbacks — reactions as buttons

**What's this?** Matrix doesn't have inline buttons like Telegram. But it has reactions (emojis). And they can be used as buttons.

**Example:**
```
Bot:  "Confirm action?"
      (bot adds ✅ and ❌ reactions)

You:  react with ✅ on the bot's message
Bot:  "✅ Done!" and removes reactions
```

**How does it work under the hood?** The bot tracks who reacted with which emoji and calls the corresponding function.

---

## Rate Limiter — ban protection

**What's this?** Matrix servers can ban you for too many requests (error 429). Rate Limiter automatically slows the bot down if the server asks.

**How does it work?**
- Everything's fine → requests go with minimal delay
- Got a 429 → automatically increase delay
- Everything's fine again → gradually speed back up

**What this means for you:** You don't need to worry about the bot spamming reactions and getting you banned. The limiter handles it all.

---

## Security — sandbox for modules

**What's this?** Community modules (not built-in) run in a restricted environment. They can't cause harm.

**What's forbidden:**
- Reading/writing files where they shouldn't
- Importing dangerous modules (`subprocess`, `socket`, `ctypes`...)
- Using `eval()`, `exec()`
- Accessing sensitive bot data

**How does it work?**
1. AST analysis before loading
2. Audit hook for all dangerous operations during runtime
3. Frozen core modules — can't be modified from community code
4. ScopedDatabase — each module only sees its own data in the DB

**This means:** You can install modules from questionable repositories with much less risk. But still use your head.

---

## Event Handler

**What's this?** Commands only react to messages. Event Handlers react to ANY Matrix event:
- Someone joined a room
- Someone left a room
- Room name was changed
- Someone was invited
- etc.

**Example:** When someone enters a room — the bot automatically greets them.

---

## TL;DR

| Concept | What it does |
|---------|-------------|
| Command | Reacts to `.command` |
| Watcher | Reacts to ANY message matching a pattern |
| FSM | Step-by-step dialog |
| Cron | Automatically on a schedule |
| Emoji Callbacks | Reactions as buttons |
| Rate Limiter | Protection against 429 |
| Security | Sandbox for third-party code |
| Event Handler | Any Matrix events |

---

**For developers:** If you want to write modules, check the [Writing Modules](writing-modules/index.md) section. Everything there comes with code examples and technical details.
