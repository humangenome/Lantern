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
redist\                        WARP software renderer + the UE4SS proxy DLL
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

Use `host-instance.ps1` to launch. It applies the Engine.ini host template with
this instance's real gameplay port, launches Grounded 2 with the WARP software
renderer, and pins the process to a fixed set of CPU cores.

A few things that are load-bearing on a box with no GPU:

- `-dx12 -WARP` go together. Unreal does not support WARP under D3D11.
- Never pass `-nullrhi`. It crashes Grounded 2's render branch.
- Copy `redist\d3d10warp.dll` next to the Grounded 2 shipping exe. The in-box
  Windows WARP does not clear Unreal's SM6 adapter check; this one does.
- Steam must be logged in on the host. A logged-out Steam exits at a re-login
  gate, which is an account state rather than a Lantern fault.
- Kill any leftover `CrashReportClient*` and `WerFault` processes before
  relaunching, or they wedge the next start.

WARP renders on the CPU, so one instance will use a whole machine if you let it.
`host-instance.ps1` bounds each instance to a disjoint block of cores derived
from its gameplay port, which is how you fit more than one instance on a box. It
refuses to start rather than overlap another instance's cores.
