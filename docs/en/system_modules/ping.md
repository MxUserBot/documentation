# PingPong — Latency Check Module

**File:** `modules/ping.py`

## Description

A simple module for measuring latency between the bot and the Matrix server. Sends a message, measures the time, edits it, and displays the result.

## Commands

### `.ping`

**Access:** OWNER

Sends a "🏓 | Pinging..." message, measures the time until the Matrix server acknowledges it, and edits it to "🏓 | Pong! 🚀 | Latency: X.XX ms".

```
.ping
# → 🏓 | Pinging...
# → 🏓 | Pong!
#    🚀 | Latency: 42.17 ms
```

## Details

- Uses `time.perf_counter()` for maximum measurement accuracy
- Measures round-trip time to the server (from send to receipt confirmation)
- Not affected by network latency between the server and the user's client

| Command | Access | Description |
|---------|--------|-------------|
| `.ping` | OWNER | Check latency |
