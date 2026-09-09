# ADR 0004 — Ship a template config + README instead of a script

- Date: 2013-09-07 (shape established in `88d6ce0`, refined across seven same-day commits)
- Status: accepted

## Context

The deliverable could have been: (a) an installer script that writes
`/etc/wpa_supplicant.conf` and installs the helpers, (b) a dotfile-style
repo users clone and symlink, or (c) a two-file repo — a template config
plus a README of copy-paste instructions. The audience is a student who
needs internet *now*, on an arbitrary distro, with root access via
`sudo`, and with credentials that must never leave their machine.

## Decision

Ship (c): `wpa_supplicant.conf` as a committed template with placeholder
credential fields, and a README whose shell functions users paste into
their own shellrc. No installer, no dependency on any language runtime,
no cloned-repo state on the user's machine beyond what they copy out of
it.

## Consequences

- Zero-dependency portability: anything with a web browser and a
  terminal can use it; nothing to execute from the repo itself.
- The README became the product — nine of eleven commits refine it,
  including the only bug fix (a markdown edit, `b6293c0`).
- Fixes don't propagate: users who copied an earlier `refresh()` kept
  the `dhclient -r` bug until they re-copied; the 2016 typo fix
  (`80771c1`) exists because copy-paste was the distribution channel.
- Sensitive by construction: since users hand-edit the config with real
  credentials, the repo itself must never contain any (ADR 0005).

## Revisit

The author explicitly invited a startup script in `e9c0557` ("I welcome
a startup script"), i.e., acknowledged the manual-copy model's ceiling —
but preferred soliciting it from users over building it himself. It was
never built.
