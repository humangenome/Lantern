# Mods on Lantern

Lantern loads mods through [UE4SS](https://github.com/UE4SS-RE/RE-UE4SS) on both
the host and the client. There are two layers:

1. **Lua mods** dropped into the UE4SS `Mods` folder and listed in `mods.txt`.
2. **Native (C++) plugins**, loaded by a small UE4SS Lua mod that calls
   `package.loadlib(path, "*")` (the same as `LoadLibrary`), which runs the DLL's
   `DllMain`. Native plugins are not loaded as UE4SS C++ mods: the
   `CppUserModBase` vtable path crashes the game on a layout mismatch.

## Where mods live

| Side | Path | Loaded by |
|------|------|-----------|
| Host | `ue4ss\Mods\` in the server package | host UE4SS, `mods.txt` |
| Client | `ue4ss\Mods\` next to the Grounded 2 executable, installed by the app | client UE4SS, `mods.txt` |

`mods.txt` lines are `<ModName> : 1` to enable. UE4SS treats mixed LF/CRLF
inconsistently, so keep `mods.txt` CRLF-terminated (the release build enforces
this for the shipped packages).

## Lantern's own mods

- `g2_sshost` (host): drives the host through the lobby into the saved or new
  world and keeps the connected players with it.
- `LanternConnect` (client): issues `open <ip>:<port>` to join, and waits for
  the game's own sign-in and main menu before it does.

## Writing your own

Match the structure of Lantern's mods: a `<ModName>/Scripts/main.lua` plus an
`enabled.txt`, then add `<ModName> : 1` to `mods.txt`. Resolve UObjects on the
game thread (`ExecuteInGameThread`) and never block the UE4SS async loop.


## Don't

- Don't post anti-cheat-evasion or game-piracy material in mod issues/PRs.
- Don't modify the retail Grounded 2 game files. Keep changes in the UE4SS
  overlay and Engine.ini so Steam updates keep working.
