# One-shot recreation prompt — IllinoisNet (Linux)

Give this prompt to a capable agent with an empty git repo; it should
recreate this project (files equivalent up to wording) without seeing
the original.

---

Recreate a tiny, documentation-first git repository called
**IllinoisNet** whose entire purpose is to help a student connect a
Linux laptop to **IllinoisNet**, the University of Illinois
Urbana-Champaign WPA2-Enterprise Wi-Fi, circa 2013, when NetworkManager
and wicd could not reliably join it. The repo contains exactly two
functional files — a template `wpa_supplicant.conf` and a `README.md` —
plus documentation (CHANGELOG.md, AGENTS.md, architectural-diary/,
prompt.md). There is no code to run from the repo; the README's shell
functions are meant to be copied into the user's `~/.bashrc` or
`~/.zshrc` and executed by the *user*, never by the repo. Never attempt
to execute any network/login command yourself.

## Goal

A user with a UIUC NetID and password reaches the internet through
IllinoisNet by: copying one config file to `/etc`, filling in their own
credentials (never committed), and running a single shell function —
with every step visible in their terminal.

## Stack (2013-era Linux; document, do not depend on)

- `wpa_supplicant` with WPA-EAP/TTLS support
- `dhclient` for DHCP; `iwconfig` to identify the wireless interface
- `network-manager` and `wicd` (both must be stopped before connecting)
- `iconv -t utf16le | openssl md4` to compute the NT password hash
- `wpa_passphrase` to generate additional WPA-PSK network blocks

## Build order (phases)

1. **Repo stub.** Initialize git; write a `README.md` with a title and
   one-line purpose ("Get IllinoisNet working on linux!!").
2. **Core artifact.** Write `wpa_supplicant.conf` with two blocks:
   - IllinoisNet: `key_mgmt=WPA-EAP`, `eap=TTLS`, `pairwise=CCMP`,
     `phase2="auth=MSCHAPV2"`, `identity="netid"`,
     `password="password"` — placeholders only.
   - An example extra network using `ssid="other_network"` and a dummy
     non-hex `psk=` filler.
3. **README instructions.** Document, in this order: copy the config to
   `/etc/wpa_supplicant.conf`; fill in NetID and password; the
   hash-generation one-liner `echo -n 'YOUR_PASSWORD_HERE' | iconv -t
   utf16le | openssl md4` with `password=hash:<hash>`; a warning that
   plaintext passwords require restrictive file permissions (`chmod
   600`); a "To add other networks" section using `wpa_passphrase`
   (interactive form preferred — arguments leak into shell history).
4. **Shell helpers.** In fenced ```bash blocks, define `internet()` and
   `refresh()` for `wlan0`, plus duplicates `internet1()`/`refresh1()`
   hard-coded to `wlan1` (chosen by checking `iwconfig`).
   `internet()` must, in order: `sudo service network-manager stop`;
   `sudo service wicd stop`; `sudo killall wpa_supplicant`;
   `sudo wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf -B`;
   `sudo dhclient -r wlan0`; `sudo dhclient wlan0`; `sudo iwconfig`.
   `refresh()` must run ONLY `sudo dhclient wlan0` — never `-r` alone
   (releasing without requesting disconnects the user; this was the
   repo's one real bug).
5. **Close with a contribution invite** ("I welcome a startup script")
   and a credit line thanking github.com/zach297 for demonstrating the
   procedure.
6. **Docs pass** (single commit): add CHANGELOG.md (all commits, newest
   first, 1–3 bullets each), AGENTS.md (commands, structure map,
   conventions, gotchas, verification), architectural-diary/ (main.md +
   numbered ADRs), and this prompt.md. Frame everything 2013-era as
   archived/legacy.

## Design decisions (all of them)

1. **EAP-TTLS + MSCHAPv2 phase 2** — matches campus RADIUS; needs only
   NetID + password (or NT hash), no client certificates. `pairwise=CCMP`.
2. **NT hash instead of cleartext in `/etc`** — store
   `password=hash:<MD4-of-UTF16LE>`; cleartext allowed but warned
   against with `chmod 600` guidance. (The hash is still a usable
   authenticator — permissions still matter.)
3. **Bypass desktop network managers** — the helpers stop
   network-manager and wicd and kill any running `wpa_supplicant`
   before starting their own, so association is deterministic and
   observable.
4. **Template + README, not a script** — nothing in the repo executes;
   users copy a config template and paste functions into their shellrc.
   Fixes propagate only when users re-copy.
5. **Placeholder credentials only** — `identity="netid"`,
   `password="password"`, dummy `psk`; real values never enter git.
6. **Duplicated `wlan1` variants over parameters** — copy-paste
   ergonomics beat DRY for the intended audience.
7. **DHCP renew = plain `dhclient`, never bare `-r`** — `dhclient -r`
   releases and disconnects.
8. **History hygiene (2026)** — commit messages in Conventional Commits
   form (`type: imperative subject` ≤72 chars, body explains why); docs
   never contain credentials, only field names and command forms.

## Data model

There is no runtime data model. The single persisted structure is the
`wpa_supplicant.conf` dialect: `network={ ... }` blocks with fields
`ssid`, `key_mgmt`, `eap`, `pairwise`, `phase2`, `identity`,
`password` (cleartext or `hash:`-prefixed NT hash), and `psk`. All
committed values are placeholders; users materialize real credentials
in `/etc/wpa_supplicant.conf` only.

## Protocols / interfaces (by name)

- **IEEE 802.1X / WPA2-Enterprise** over the IllinoisNet SSID, keyed by
  `key_mgmt=WPA-EAP` with **CCMP** pairwise cipher.
- **EAP-TTLS** as the outer EAP method; **MSCHAPv2** as the phase-2
  inner auth (`phase2="auth=MSCHAPV2"`), verified by the campus
  **RADIUS** infrastructure against NetID credentials.
- **DHCP** via `dhclient` (release with `-r`, request with no flag).
- Shell interface: functions `internet`, `internet1`, `refresh`,
  `refresh1` (documented in README only).

## Acceptance criteria

1. Repo contains `wpa_supplicant.conf` and `README.md` as the only
   functional files; everything else is documentation.
2. The IllinoisNet block uses exactly WPA-EAP / TTLS / CCMP /
   MSCHAPv2 with placeholder `identity` and `password`.
3. The README documents, in copy-pasteable fenced bash blocks: config
   copy to `/etc`, NT-hash generation, permissions warning,
   `wpa_passphrase` section, and all four shell functions in the exact
   command order from the build order (phase 4).
4. `refresh()` and `refresh1()` contain no bare `dhclient -r`.
5. No real or realistic credential appears anywhere in the repo —
   placeholders and field names only.
6. All shell/network commands appear only inside fenced code blocks or
   as prose; the repo itself contains nothing executable and the agent
   never runs the commands.
7. Docs set exists: CHANGELOG.md, AGENTS.md, README.md,
   architectural-diary/main.md + numbered decisions, prompt.md — with
   the 2013 content framed as archived/legacy.
