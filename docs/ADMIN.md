# Lantern server admin guide

Generic admin guide for running a self-hosted LanternServer. Managed-hosting
customers should use their host's panel instead — these are the raw controls.

## Server package layout

```
LanternServer\
├── LanternServer.exe          the host supervisor (RCON, query, HTTP API)
├── appsettings.json           per-instance config
ue4ss\                         host-side UE4SS layout (g2_sshost host-start mod)
engine-ini\                    Engine.ini host/client templates
warp\                          WARP redist notes
host-instance.ps1              launch + CPU-affinity helper
steam_appid.txt
```

## Settings (`LanternServer\appsettings.json`)

The `Lantern` block carries the per-instance config. At minimum set:

- `InstanceId` — a stable id for this instance
- `GameplayPort` / query / RCON / HTTP ports (see ports below)
- `RconPassword` — required to use RCON
- `ServerName` — shown in the server query / browser

`Mods` settings nest **under** the `Lantern` block, not at the top level.

## Ports

| Offset | Default | Purpose | Proto |
|--------|---------|---------|-------|
| +0 | 7777 | Gameplay (the join port) | UDP |
| +1 | 7778 | Control / IPC identifier (local only) | — |
| +2 | 7779 | Server query (A2S) | UDP |
| +3 | 7780 | RCON | TCP |
| +4 | 7781 | Admin HTTP API | TCP |

Open/forward each externally-reachable port (+0, +2, +3, +4). The control port
(+1) is a local IPC identifier and needs no firewall rule. The gameplay UDP
port in particular needs a Windows Defender inbound allow rule, or players can't
reach the listen socket.

## RCON commands

LanternServer exposes Source RCON on the RCON port:

- `help` — list commands
- `status` — server status
- `players` — connected players
- `ping` — liveness
- `save snapshot` — take a world snapshot
- `save list` — list saved worlds
- `save restore <id>` — roll the world back to a snapshot

## Server query

LanternServer answers Source A2S on the query port, so any standard server-list
tool, monitor, or bot can read status and player count.

## Running

Use `host-instance.ps1` to launch — it applies the Engine.ini host template,
launches Grounded 2 with the WARP args, and pins the process to its CPU-core
mask. See `RUNTIME.md` for the full boot recipe and density model.
