# HOW DOES THIS USERBOT WORK?

## Concept
A userbot is a bot that runs under your Matrix account. All commands are executed on behalf of your account.

## Startup Sequence

```
1. Start
       ↓
2. MXUS (Security) — air-gapped firewall without DB
       ↓
3. RocksDB initialization (key from MXUS)
       ↓
4. FastAPI web server (login via browser)
       ↓
5. Wait for authorization
       ↓
6. Matrix Client with token
       ↓
7. OlmMachine (crypto, encryption)
       ↓   hook on SAS verification
8. Wait for network (whoami)
       ↓
9. FULL Security initialization (DB + client)
       ↓
10. Log room [LOGS]
       ↓
11. Module loading (core + community)
       ↓
12. Event handlers (messages, reactions, invites...)
       ↓
13. Start sync  ← rate_limit_protect enabled
       ↓
14. 🚀 READY
```


## How is a command processed?

Say you type `.ping`:

```
1. Matrix sends ROOM_MESSAGE
       ↓
2. message_cb() catches it
       ↓
3. Checks: timing, ignore_ids, not empty
       ↓
4. Decrypt if needed [ee2e]
       ↓
5. Starts with prefix "."?
       ↓
6. Look up ".ping" in command_registry
       ↓
7. check_access(): who invoked?
       ↓
8. Module config check (required fields)
       ↓
9. Argument parsing (Pydantic / types / raw)
       ↓
10. Module function execution
       ↓
11. Reply via utils.answer()
```

---

## MXBotInterface — safe wrapper

Community modules receive **not** the bot itself, but `MXBotInterface` — a wrapper:
- `client` — Matrix API
- `fsm` (with `_processed_events` for deduplication), `sas_verifier`, `security`, `_prefixes`
- **No** access to: `crypto`, `api`, and other dangerous calls.

---

## Database: RocksDB

- All values are encrypted with Fernet
- Key at `mxu.rocksdb/.mxu.key` with permissions 600
- Format: `"owner:key"` (owner = module name or "core")
- Community modules can only see their own keys. This prevents them from stealing other modules' data.

---

## Module System

- **Core** — built-in, `src/mxuserbot/modules/` — frozen from changes
- **Community** — from repositories, `community/` — sandboxed

More about module development in `0-key-concepts.md`.
