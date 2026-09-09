# ADR 0005 — Commit only placeholder credentials

- Date: 2013-09-07 (implicit from `88d6ce0` onward; restated 2026-09-08)
- Status: accepted, enforced

## Context

The template `wpa_supplicant.conf` must show every field a user needs to
fill — `identity`, `password`, and an example `psk` for other networks.
Those fields are, by construction, where campus credentials would go.
Anything real committed here would be a public leak (the repo is public
on GitHub) of a NetID, a password, an NT hash usable for
pass-the-hash against campus RADIUS, or a home Wi-Fi PSK.

## Decision

The committed file contains only obvious placeholders:

- `identity="netid"` — generic username placeholder, never a real NetID;
- `password="password"` — literal placeholder string;
- the example PSK block uses a dummy, non-hex filler string that
  `wpa_passphrase` would never emit.

Real values exist only in the user's `/etc/wpa_supplicant.conf` copy,
which the README instructs to permission `600`. Documentation (README,
this diary, `prompt.md`) may reference field *names* and command *forms*
only — never stored credential values.

## Consequences

- The repo is safe to fork, clone, archive, and embed in datasets
  regardless of who holds the credentials it teaches users to configure.
- Verification rule added to AGENTS.md: any doc or config change is
  scanned for credentials-adjacent strings before commit.
- No `.gitignore`-style technical enforcement exists (no tooling in the
  repo), so this is a convention backed by review, not by CI.

## Revisit

Confirmed during the 2026-09-08 documentation pass: all committed values
are placeholders; no secrets found in any commit or doc.
