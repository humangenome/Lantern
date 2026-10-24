# Lantern server admin guide

Settings, ports and RCON commands of a LanternServer. Self-hosting is not supported yet (see Running), so these
are the raw controls behind a rented server; your host's panel sets them for you.

## Server package layout

```
LanternServer\
├── LanternServer.exe          the host supervisor (RCON, query, HTTP API)
├── appsettings.json           per-instance config
ue4ss\                         host-side UE4SS layout (g2_sshost host-start mod)
engine-ini\                    Engine.ini host/client templates
redist\                        WARP software renderer + the UE4SS proxy DLL
warp\                          WARP redist notes
host-instance.ps1              launch + CPU-affinity helper
steam_appid.txt
```

## Settings (`LanternServer\appsettings.json`)

The `Lantern` block carries the per-instance config. At minimum set:

- `InstanceId`: a stable id for this instance
- `GameplayPort` / query / RCON / HTTP ports (see ports below)
- `RconPassword`: required to use RCON
- `ServerName`: shown in the server query / browser

`Mods` settings nest **under** the `Lantern` block, not at the top level.

## Ports

| Offset | Default | Purpose | Proto |
|--------|---------|---------|-------|
| +0 | 7777 | Gameplay (the join port) | UDP |
| +1 | 7778 | Control / IPC identifier (local only) | none |
| +2 | 7779 | Server query (A2S) | UDP |
| +3 | 7780 | RCON | TCP |
| +4 | 7781 | Admin HTTP API | TCP |

Open/forward each externally-reachable port (+0, +2, +3, +4). The control port
(+1) is a local IPC identifier and needs no firewall rule. The gameplay UDP
port in particular needs a Windows Defender inbound allow rule, or players can't
reach the listen socket.

## RCON commands

LanternServer exposes Source RCON on the RCON port:

- `help`: list commands
- `status`: game and runtime state
- `players`: connected players
- `ping`: liveness
- `save snapshot`: take a world snapshot
- `save list`: list recent snapshots

Restoring a snapshot is not an RCON command. Use the admin HTTP API on the HTTP
port (`GET /api/v1/snapshots` to list, `POST /api/v1/snapshots/<id>/restore` to
roll back, `POST /api/v1/snapshots/import-restore` to restore an uploaded world)
or the app's World window. Admin routes are HMAC-signed with a key derived
from the RCON password.

## Server query

LanternServer answers Source A2S on the query port, so any standard server-list
tool, monitor, or bot can read status and player count.

## Running

Self-hosting is not supported yet. The server package runs only inside the per-server setup a hosting panel does
around it: it puts the UE4SS runtime and the host mod into the game folder, writes the game's Engine.ini and
launch settings for that server, starts the game with the environment the host mod reads, and watches it.
`host-instance.ps1` in the package does none of that; it only launches the game and pins it to two cores. Started
that way (tested 2026-10-19), Grounded 2 never opened a server port and LanternServer reported the game as down.
