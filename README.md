# Minecraft AFK Bot

A robust 24/7 AFK bot for Minecraft servers. Keeps a bot account online indefinitely, looks human-ish to avoid anti-AFK kicks, auto-reconnects, fights mobs, sleeps, chats, and serves a live status dashboard — all from a single `index.js` file.

## Features

- **24/7 uptime** — runs in any Node.js environment (local, Railway, Render, etc.)
- **Auto-reconnect** — exponential backoff with jitter, throttle-aware extended delays
- **Crash recovery** — catches uncaught exceptions/unhandled rejections and reconnects on connection errors
- **Duplicate-login guard** — detects when another session holds the account and waits minutes before retrying
- **Username auto-fix** — trims invalid characters and truncates names longer than 16 chars
- **Anti-AFK** — random arm swings, hotbar switches, moves, sneaks ("teabag"), random looks
- **Movement modes** — circle walking, random jumping, look-around, or navigate to a set coordinate
- **Combat** — attacks nearby mobs, locks targets, and auto-eats when hungry
- **Auto-auth** — detects `/login` / `/register` prompts and answers them automatically
- **Chat** — responds to greetings, handles `!tp` for whitelisted users, logs chat, and cycles preset messages
- **Bed module** — finds and sleeps in a bed at night
- **Web dashboard** — live status, uptime, coordinates, plus a setup guide
- **Readable kick reasons** — translates raw kick JSON into plain text (whitelist, online-mode, duplicate login, throttled, etc.)
- **Fancy console** — all output rendered inside an aligned, color-coded box; long lines wrap to the next row instead of being cut off

## Requirements

- Node.js 16+
- npm
- A Minecraft server with **cracked / offline mode** enabled (or a premium account with password set)

## Installation

```bash
npm install
```

## Configuration

Edit `settings.json` before running:

```json
{
  "bot-account": {
    "username": "BotUser 01",
    "password": "",
    "type": "offline"
  },
  "server": {
    "ip": "ip.aternos.me",
    "port": 00000,
    "try-creative": false
  }
}
```

### `bot-account`
| Key | Type | Description |
|---|---|---|
| `username` | string | Bot username. Must be 3-16 valid chars; anything invalid or too long is auto-fixed before connecting. Overridden by `BOT_USERNAME` env var. |
| `password` | string | Mojang/Microsoft account password (only for premium servers). Leave empty for offline mode. |
| `type` | string | `offline` for cracked servers, `microsoft` / `mojang` for premium. |

### `server`
| Key | Type | Description |
|---|---|---|
| `ip` | string | Server address. |
| `port` | number | Server port. |
| `try-creative` | boolean | Attempt `/gamemode creative` after spawn (requires OP). |

### `position`
| Key | Type | Description |
|---|---|---|
| `enabled` | boolean | Pathfind to the coordinates below on spawn (disabled if circle-walk is on). |
| `x` / `y` / `z` | number | Target block coordinates. |

### `utils`
| Key | Type | Description |
|---|---|---|
| `auto-auth.enabled` | boolean | Auto-answer auth prompts. |
| `auto-auth.password` | string | **Set this** — the `/login` / `/register` password. |
| `anti-afk.enabled` | boolean | Run anti-AFK actions. |
| `anti-afk.sneak` | boolean | Stay sneaked permanently. |
| `chat-messages.enabled` | boolean | Send preset chat messages. |
| `chat-messages.repeat` | boolean | Loop messages forever. |
| `chat-messages.repeat-delay` | number | Seconds between repeated messages. |
| `chat-messages.messages` | string[] | Messages to send. |
| `chat-log` | boolean | Print server chat to the console. |
| `auto-reconnect` | boolean | Reconnect after disconnects. |
| `auto-reconnect-delay` | number | Base reconnect delay in ms. |
| `max-reconnect-delay` | number | Maximum backoff delay in ms. |

### `movement`
| Key | Type | Description |
|---|---|---|
| `enabled` | boolean | Master switch for movement modules. |
| `circle-walk.enabled` | boolean | Walk a circle around the current spot. |
| `circle-walk.radius` | number | Circle radius in blocks. |
| `circle-walk.speed` | number | Milliseconds between path updates. |
| `look-around.enabled` | boolean | Randomly look around. |
| `look-around.interval` | number | Milliseconds between looks. |
| `random-jump.enabled` | boolean | Randomly jump (ignored while circle-walk is on). |
| `random-jump.interval` | number | Milliseconds between jumps. |

### `modules`
| Key | Type | Description |
|---|---|---|
| `avoidMobs` | boolean | Back away from mobs/players closer than 5 blocks. |
| `combat` | boolean | Fight mobs within 4 blocks (uses `combat` section). |
| `beds` | boolean | Enable bed module (uses `beds` section). |
| `chat` | boolean | Enable chat responses (uses `chat` section). |
| `console-commands` | boolean | Read commands from stdin while running. |

### `combat`
| Key | Type | Description |
|---|---|---|
| `attack-mobs` | boolean | Attack nearby mobs. |
| `auto-eat` | boolean | Eat food automatically when hunger < 14. |

### `beds`
| Key | Type | Description |
|---|---|---|
| `pick-up-day` | boolean | *(reserved)* |
| `place-night` | boolean | Sleep in a nearby bed at night. |

### `chat`
| Key | Type | Description |
|---|---|---|
| `respond` | boolean | Reply to greetings and handle `!tp`. |
| `tpWhitelist` | string[] | Minecraft usernames allowed to use `!tp`. |

## Environment Variables

| Variable | Description |
|---|---|
| `PORT` | Web dashboard port (default `5000`). |
| `BOT_USERNAME` | Overrides `bot-account.username`. |
| `BOT_PASSWORD` | Overrides `bot-account.password` (premium auth). |
| `BOT_AUTH_PASSWORD` | Overrides the `/login` / `/register` password. |
| `RENDER_EXTERNAL_URL` | Render public URL — enables self-ping keep-alive. |
| `RAILWAY_STATIC_URL` | Railway public URL — enables self-ping keep-alive. |

## Usage

```bash
npm start
# or
node index.js
```

The bot prints a startup banner and connects automatically. Press `Ctrl+C` (or send `SIGTERM`) to shut down gracefully after 5 seconds.

### Console commands

With `modules.console-commands` enabled (and stdout piped / non-TTY), type lines in the terminal:

| Input | Action |
|---|---|
| `say <text>` | Chat as the bot. |
| `cmd <command>` | Run a Minecraft server command (e.g. `cmd tp me`). |
| `status` | Show connection status and uptime. |
| anything else | Send the line directly as chat. |

## Web Dashboard

A live dashboard is served at `http://localhost:5000/` (auto-increments the port if taken).

- `/` — dashboard with status, uptime, and coordinates (auto-refreshes every 5s)
- `/health` — JSON status for monitoring
- `/ping` — heartbeat endpoint (for UptimeRobot and self-ping)
- `/tutorial` — built-in hosting/setup guide

## Keeping It Alive

To run 24/7 you need a host that never sleeps, plus a monitor that wakes it:

1. Deploy the repo to a platform like **Railway** or **Render** (they auto-detect Node.js).
2. Set the platform's public URL env var (`RAILWAY_STATIC_URL` / `RENDER_EXTERNAL_URL`) so the bot self-pings every 10 minutes.
3. Optionally add an **UptimeRobot** HTTP monitor against `/ping` to guarantee activity.

## Project Structure

```
.
├── index.js            # Entire bot (connection, modules, dashboard, UI)
├── settings.json       # All bot configuration
├── package.json        # Dependencies and start script
├── dependabot.yml      # Daily dependency updates
└── launcher_accounts.json  # Auto-generated session cache (don't edit)
```

## How It Works (high level)

1. **Startup** — loads config, starts the Express dashboard server, and shows a boxed banner.
2. **Connect** — creates a mineflayer bot, loads the pathfinder plugin, and sets a 150s spawn timeout.
3. **Spawn** — marks connected, initializes modules (`initializeModules`), optionally switches to creative.
4. **Modules** — each enabled config section starts its own intervals or event listeners.
5. **Reconnect** — on kick/end/error/fatal, schedules reconnection with exponential backoff. `duplicate_login` kicks get a much longer delay so a stray session can drop first.
6. **Keep-alive** — if a public URL env var exists, self-pings `/ping` every 10 minutes.
7. **Console UI** — every `stdout`/`stderr` write is intercepted and rendered inside an aligned box. Long lines wrap to additional rows rather than truncating, and kick reasons are translated to readable text.

## Troubleshooting

- **`duplicate_login` kick** — another instance of the bot is running (or the account is in use elsewhere). Ensure only one `node index.js` process exists. The bot auto-detects this and waits progressively longer (5+ min) before retrying.
- **`not whitelisted` kick** — the whitelist doesn't contain the exact (case-sensitive) name the bot uses. Add the fixed bot name to the server whitelist.
- **`You need to log in` (unverified_username) kick** — the server is in online mode. Use cracked/offline mode, or set a real premium account in `bot-account`.
- **Bot never spawns** — check `server.ip` / `server.port`, and that the server uses offline mode (or you set a password as the account type).
- **`disconnect.generic_reason` kick** — usually an invalid username (over 16 chars). The bot now truncates it automatically; make sure the server's whitelist matches the fixed name.
- **Auth prompt ignored** — make sure `utils.auto-auth.password` is set (also works via `BOT_AUTH_PASSWORD`).
- **Port already in use** — the dashboard auto-advances to the next free port, so this is harmless.

## License

MIT
