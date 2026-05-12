# HOW IT WORKS

Basically, in simple terms.

## You start the bot

1. First, security is checked (firewall)
2. Database connects (RocksDB)
3. Web server starts — so you can log in through the browser
4. Waits for you to authorize
5. Connects to Matrix
6. Crypto connects (OlmMachine)
7. Modules load: first built-in, then yours from repositories
8. Starts listening for new messages

## When you type a command

Say you type `.ping`:

1. Matrix sends a "new message" event
2. The bot catches this event
3. Decrypts if needed (if the room is encrypted)
4. Sees it starts with `.`
5. Looks for the matching command
6. Checks permissions (are you OWNER? SUDO? or just someone?)
7. If all good — executes the command
8. Replies via `utils.answer()`

## Modules

There are two types:

- **Core** — built-in, ship with the bot. You can't change them, they're "frozen".
- **Community** — ones you install from repositories. Run in a sandbox, can't do harm.

## If something's not clear

Check out [Key Concepts](key-concepts.md) — everything explained in simple terms.

If you're a developer and want to write modules, head over to [Writing Modules](writing-modules/index.md).
