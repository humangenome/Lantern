# Security Policy

## Reporting a vulnerability

If you have found a security issue in Lantern (the server, the in-game runtime, the launcher, or the admin API), please **do not** open a public GitHub issue.

Open a private security advisory at https://github.com/HumanGenome/Lantern/security/advisories/new. That is the preferred channel.

Include:

- A description of the vulnerability
- Steps to reproduce
- Affected component (server / runtime / launcher / API)
- Lantern version
- Whether the issue is currently being exploited

Reports are acknowledged within 72 hours and triaged within 7 days.

## Scope

In scope:

- Remote code execution or unauthenticated takeover of `LanternServer.exe`
- Authentication bypass on the join handshake, RCON, or the admin HTTP API
- Anything a connected client can do to write arbitrary files on the host
- Privilege escalation through the in-game runtime

Out of scope:

- Vulnerabilities in the host machine itself — those belong to whoever runs it
- Vulnerabilities in retail Grounded 2 — report those to the game's publisher
- Vulnerabilities in third-party mods running on Lantern
- Cheating and anti-cheat concerns. Lantern does not provide anti-cheat.
