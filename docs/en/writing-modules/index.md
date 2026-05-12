# Writing Modules

Everything you need to write your own modules.

**For regular folks:** If you're not planning to write modules, read [Key Concepts](../key-concepts.md) — it explains FSM, Watcher, Cron and more in simple terms.

---

## Getting Started

If you're a beginner, start with [Basics](basics.md). It covers the mandatory parts of any module: Meta, strings, `@loader.tds`.

---

## Sections

| Section | What it's about |
|---------|----------------|
| [Basics](basics.md) | Minimal module, Meta, strings, lifecycle |
| [Commands](commands.md) | `@loader.command`, argument parsing, config |
| [Answer Utils](answer-utils.md) | `utils.answer`, HTTP requests, media |
| [Watcher, Cron, Events](watcher-cron-events.md) | Auto-reactions, scheduled tasks, event handlers |
| [FSM](fsm.md) | Step-by-step dialogs, Pydantic validation |
| [Emoji Callbacks](emoji-callbacks.md) | Reactions as buttons |
| [Security](security.md) | Access levels, sandbox, audit hook |
| [ZIP Modules](zip-modules.md) | Packaging modules as zip packages |
| [Distribution](distribute.md) | How to share your module with others |
| [Utils Reference](utils-reference.md) | All `utils.*` functions |

---

## Optional reading

- [SAS Verification](sas-verification.md) — device verification from the bot
