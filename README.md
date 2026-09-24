# zomboid-server

A Project Zomboid **Build 42** dedicated server with a web panel to run it from
anywhere, in English or Spanish.

- **Server:** start, stop and restart with in-game countdown warnings. Update or
  verify the game, pick the Steam branch and Java memory.
- **Configuration:** every server and world (sandbox) setting as a form with the
  game's own descriptions, a raw editor for admins, history with diff and
  one-click revert.
- **Backups:** manual and scheduled, restore with undo, download and upload.
- **Reset:** new world; new world and accounts; or factory. A backup is always
  taken first.
- **Players:** who's online, join history, kick, ban, access levels, whitelist,
  broadcast, live log and console.
- **Mods:** add by Workshop link or collection. Dependencies and load order are
  worked out for you, and update checks run on a schedule.
- **Schedules:** daily restarts with a quiet backup, periodic backups, and
  automatic game and mod updates when nobody is playing.
- **Discord:** optional messages for server, player, backup and security events.
- **Access:** accounts with roles (viewer, operator, admin, owner), 2FA,
  sessions and an audit log.

## How it fits together

```
 friends ── UDP 16261-16262 ──▶ pz      game server + agent (owns the process, RCON, steamcmd)
                                  ▲ backend network (agent API, token-protected)
 browser ── HTTPS 8443 ──▶ caddy ──▶ panel   API + web UI, users, config, backups, schedules
```

Three containers in one Compose project, `zomboid`. Only the game ports and the
panel's HTTPS port are published. RCON and the agent API never leave the
internal network.

## Run it

```bash
node scripts/init-env.mjs        # creates .env with fresh secrets; then review it
docker compose up -d --build
```

Open `https://localhost:8443` and accept the certificate warning (it's
self-signed; see [security](docs/security.md)). Sign in as `owner` with the
`PANEL_OWNER_PASSWORD` from `.env`. You'll be asked to pick a new password and
set up 2FA. Then press **Start**; the first start downloads the game (a few GB).

- [Windows runbook](docs/runbook-windows.md): Docker on Windows, router, day-to-day
  operation, troubleshooting.
- [Linux runbook](docs/runbook-linux.md): moving to a VPS or Linux box.
- [Security](docs/security.md): what's protected and how, plus what to do
  when something goes wrong.

## Develop

```bash
npm ci
node scripts/dev.mjs             # agent + fake game server + panel + Vite on http://localhost:5173
bash scripts/verify.sh           # lint, typecheck, tests, audit, line endings
```

`dev.mjs` runs everything against `tools/fake-pz`, a stand-in that speaks the
same log lines and RCON as the real server. So the whole panel works without
the 5 GB download. Sign in as `owner` / `dev-owner-password`.

| Path | What |
|---|---|
| `packages/shared` | Agent API types and the role/permission matrix |
| `packages/pz-formats` | Pure parsers for PZ files (ini, sandbox Lua, mod.info, VDF, RCON, logs), tested against `fixtures/b42` |
| `packages/agent` | Runs inside the `pz` container: process supervisor, RCON, steamcmd |
| `packages/panel` | Fastify API, SQLite, backups, scheduler, Discord |
| `packages/web` | React UI (Mantine), EN/ES |
| `docs/verification-log.md` | B42 behaviour measured on a real server |

Conventions are in [CONTRIBUTING.md](CONTRIBUTING.md). The version lives in `VERSION`.

## License

[MIT](LICENSE): free to use, change and share. The software comes as is,
without any warranty: you run it at your own risk, including the game server
you expose to the internet and the data it holds.

`fixtures/b42/` contains output captured from the Project Zomboid dedicated
server and `mod.info` files from public Steam Workshop mods, kept only as test
data. They belong to their owners and are not covered by the MIT license.
This project is not affiliated with The Indie Stone or Valve.
