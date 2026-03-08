# telepi

A small, opinionated, slightly paranoid bridge between Telegram and [pi](https://github.com/badlogic/pi-mono).

License: [MIT](./LICENSE)

## What this is

This repo is a **single-owner remote admin bridge** for talking to **pi** through Telegram.

That is the whole trick:

- **pi** is the agent
- **this repo** is the Telegram-shaped door
- a few locks were added so the door is not completely feral

No giant platform. No orchestration theme park. No "AI operating system for synergy-driven workflows." Just a Telegram bot wired into pi.

## What this is not

- not a SaaS
- not a multi-user chat platform
- not an enterprise product
- not guaranteed to be a good idea

## Vibe-coded disclaimer

This project was **vibe coded**.

Which means:

- it works for me
- it may work for you
- it may also become a tiny goblin at an inconvenient time

There is **no warranty** and **no promise of fitness for any purpose**.
Please review the code, restrict the workspace, and do not point this at anything you cannot afford to break.

## Dependency: pi

This bridge depends on **pi** and uses the **pi SDK**.

- pi repo: https://github.com/badlogic/pi-mono
- npm package: https://www.npmjs.com/package/@mariozechner/pi-coding-agent

You need pi installed, configured, and authenticated separately.
This bridge does **not** replace pi. It just gives pi a Telegram handle.

Examples:

- `pi /login`
- credentials stored in `~/.pi/agent`

If pi is not working locally, this bridge will not magically become wise through suffering.

## Features

- Telegram bot connection
- pi-powered responses through the pi SDK
- persistent pi session history per Telegram chat
- locked by default
- owner-only by Telegram user ID
- private-chat-only by default
- temporary unlock with **TOTP** or shared secret
- auto-lock after a configurable timeout
- audit logging
- optional owner alerts on denied attempts
- built-in TUI manager
- CLI manager commands

## TUI preview

<p align="center">
  <img src="./docs/media/tui-screenshot.png" alt="telepi TUI screenshot" width="900" />
</p>

Animated preview: [`docs/media/tui-demo.gif`](./docs/media/tui-demo.gif)

## Security model

This version is designed for **one owner only** and **remote admin-style access**.

It is:

- locked by default
- owner-only by Telegram user ID
- private-chat-only by default
- unlockable with **TOTP** or a shared secret
- auto-locking after a configurable timeout
- able to alert you on unauthorized attempts
- able to keep an audit log

In other words: simple, but trying not to be reckless.

## Project layout

- `src/index.mjs` — main bridge process
- `src/manage.mjs` — CLI manager
- `src/manager-lib.mjs` — shared runtime and config helpers
- `src/tui.mjs` — terminal UI manager
- `.env.example` — configuration template
- `bridge.out` — main runtime output log
- `logs/audit.log` — JSON-lines audit log
- `data/sessions/<chat-id>/...` — persistent pi session history

## Installation

### Option A: install from npm

```bash
npm install -g @lpgn/telepi
mkdir -p ~/telepi
cd ~/telepi
cp "$(npm root -g)/@lpgn/telepi/.env.example" .env
```

Then edit `.env`.

By default, `telepi` uses the current working directory for `.env`, `data/`, `logs/`, `run/`, and generated `systemd/` files.
If you want to keep those files somewhere else, set `TELEPI_HOME=/path/to/telepi-home` before running it.

### Option B: run from source

```bash
git clone https://github.com/yourname/telepi.git
cd telepi
npm install
cp .env.example .env
```

Then edit `.env`.

Minimum useful config:

```env
TELEGRAM_BOT_TOKEN=your_bot_token
OWNER_TELEGRAM_USER_ID=your_numeric_telegram_user_id
UNLOCK_METHOD=totp
UNLOCK_TOTP_SECRET=your_base32_secret
```

### 4. Make sure pi is installed and authenticated

See:

- https://github.com/badlogic/pi-mono

## Configuration notes

### Recommended: TOTP unlock

```env
UNLOCK_METHOD=totp
UNLOCK_TOTP_SECRET=JBSWY3DPEHPK3PXP
```

Use your own base32 secret and add it to an authenticator app.
Do **not** use the example secret in production unless you enjoy improvisational security.

### Alternative: shared secret unlock

```env
UNLOCK_METHOD=secret
UNLOCK_SHARED_SECRET=replace_with_a_long_random_secret
```

### Important config values

- `OWNER_TELEGRAM_USER_ID` — only this Telegram user is allowed
- `OWNER_CHAT_ID` — optional extra lock to one specific chat
- `ALLOW_PRIVATE_CHATS_ONLY` — reject groups, supergroups, and channels
- `UNLOCK_TTL_MINUTES` — auto-lock timeout
- `PI_WORKSPACE_DIR` — where pi will operate
- `PI_AGENT_DIR` — where pi config/auth lives
- `PI_MODEL_PROVIDER` / `PI_MODEL_NAME` — optional fixed model override
- `PI_THINKING_LEVEL` — optional thinking level override
- `UNLOCK_STATE_FILE` — optional persisted unlock state file
- `AUDIT_LOG_FILE` — audit log location

## Usage

### Start the bridge

```bash
telepi
```

You can still run it locally from the repo with:

```bash
npm start
```

### Use the TUI manager

```bash
telepi-tui
```

Or from the repo:

```bash
npm run tui
```

The TUI is now organized into three sections:

- **Setup** — wizard, settings, unlock secret tools, TOTP export, systemd file generation, config tests
- **Bridge** — status, start, stop, restart
- **Logs** — view and clear bridge/audit logs

Useful keys:

- `Enter` — open selected section or run selected action
- `Esc` — go back
- `1` — jump to Setup
- `2` — jump to Bridge
- `3` — jump to Logs
- `r` — refresh
- `PgUp` / `PgDn` — scroll details or logs
- `q` — quit

### Use the CLI manager

```bash
telepi-manage status
telepi-manage start
telepi-manage stop
telepi-manage restart
telepi-manage logs bridge
telepi-manage logs audit
```

Or from the repo:

```bash
npm run bridge:status
npm run bridge:start
npm run bridge:stop
npm run bridge:restart
npm run bridge:logs
npm run bridge:audit
```

### Use it from Telegram

Commands:

- `/status` — show whether the bot is locked
- `/unlock <code>` — unlock agent access temporarily
- `/lock` — lock immediately
- `/clear` — clear the current pi session, only when unlocked

Normal text prompts are forwarded to pi **only while unlocked**.

## How locking works

- the bot starts **locked**
- while locked, free-text prompts are refused
- `/unlock <code>` unlocks it for `UNLOCK_TTL_MINUTES`
- after the timeout, it auto-locks again

So yes, it is basically a remote control for pi with a dead-man switch.
Or at least a mildly anxious-man switch.

## Alerts and audit log

If `ALERT_OWNER_ON_DENIED=true`, denied attempts generate a Telegram alert to the owner chat.

Audit events are appended as JSON lines to `AUDIT_LOG_FILE`.

Typical events include:

- `DENIED_USER`
- `DENIED_CHAT_TYPE`
- `UNLOCK_SUCCESS`
- `UNLOCK_FAILURE`
- `PROMPT_START`
- `PROMPT_END`
- `PROMPT_ERROR`

## Security notes

Please do not treat "it has a lock" as equivalent to "it is safe."

Recommended precautions:

- enable Telegram 2FA on your account
- keep the bot token secret
- restrict permissions on `.env`, `logs/`, `data/`, and `~/.pi/agent`
- narrow `PI_WORKSPACE_DIR` as much as possible
- set `OWNER_CHAT_ID` if you want to pin access to one specific chat
- rotate secrets if they ever appear in chat history, shell history, screenshots, or the internet doing internet things

## systemd example

The repo includes a generic service template at:

- `systemd/telepi.service.example`

Your own machine-specific local service file should live at:

- `systemd/telepi.service`

That local file is gitignored on purpose.
The TUI can generate it for your machine.

If you want to install it system-wide, copy the generated or template file to:

- `/etc/systemd/system/telepi.service`

Example:

```ini
[Unit]
Description=telepi
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=youruser
WorkingDirectory=/opt/telepi
EnvironmentFile=/opt/telepi/.env
ExecStart=/usr/bin/env telepi
Restart=always
RestartSec=5
NoNewPrivileges=true
PrivateTmp=true
ProtectControlGroups=true
ProtectKernelTunables=true
ProtectKernelModules=true
LockPersonality=true
RestrictSUIDSGID=true
UMask=0077

[Install]
WantedBy=multi-user.target
```

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now telepi
sudo systemctl status telepi
```

## Final warning, but with affection

This repo is small on purpose.
That is a feature.

If you want a big framework, this is not that.
If you want a compact Telegram-to-pi bridge with a few security rails and a manager TUI, that is exactly what this is.

Use it, fork it, improve it, or laugh at it.
But please do so responsibly.
