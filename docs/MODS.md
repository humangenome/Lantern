# Mods on Lantern

Lantern loads mods through [UE4SS](https://github.com/UE4SS-RE/RE-UE4SS) on both
the host and the client. There are two layers:

1. **Lua mods** dropped into the UE4SS `Mods` folder and listed in `mods.txt`.
2. **Native (C++) plugins** loaded the Beacon way — a small UE4SS Lua mod calls
   `package.loadlib(path, "*")` (== `LoadLibrary`), which fires the DLL's
   `DllMain`. Native plugins are NOT loaded as UE4SS C++ mods (the
   `CppUserModBase` vtable path crashes the game on a layout mismatch).

## Where mods live

| Side | Path | Loaded by |
|------|------|-----------|
| Host | `ue4ss\Mods\` (from `dist/ue4ss-server`) | host UE4SS, `mods.txt` |
| Client | `ue4ss\Mods\` (from `dist/ue4ss-client`) | client UE4SS, `mods.txt` |

`mods.txt` lines are `<ModName> : 1` to enable. UE4SS treats mixed LF/CRLF
inconsistently — keep `mods.txt` CRLF-terminated (the release workflow enforces
this for the host package).

## Lantern's own mods

- `g2_sshost` (host) — issues `open <map>?listen` to stand up the listen world.
- `LanternConnect` (client) — issues `open <ip>:<port>` to join.

## Writing your own

Match the structure of Lantern's mods: a `<ModName>/Scripts/main.lua` plus an
`enabled.txt`, then add `<ModName> : 1` to `mods.txt`. Resolve UObjects on the
game thread (`ExecuteInGameThread`) and never block the UE4SS async loop.

A `Lantern.*` Lua ModKit API (logging, commands, players, events, chat, HTTP,
game-thread work) is planned and will be documented here when it lands.

## Don't

- Don't post anti-cheat-evasion or game-piracy material in mod issues/PRs.
- Don't modify the retail Grounded 2 game files — keep changes in the UE4SS
  overlay and Engine.ini so the Steam auto-update path stays intact.
