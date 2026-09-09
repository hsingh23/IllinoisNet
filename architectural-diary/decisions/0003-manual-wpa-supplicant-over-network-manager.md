# ADR 0003 — Bypass NetworkManager/wicd with manual wpa_supplicant + dhclient helpers

- Date: 2013-09-07 (commit `88d6ce0`; `wlan1` variants in `86440c7`; bug fixed in `b6293c0`)
- Status: accepted, historical (obsolete on modern distros)

## Context

The problem this repo exists to solve: on 2013 Linux desktops,
NetworkManager and wicd would fail on IllinoisNet's WPA2-Enterprise or
interfere with any manual attempt — a running `wpa_supplicant` instance
they own blocks the user's own. Two options: fight the desktop stacks'
configuration GUIs (which hid the EAP fields or mis-set them), or take
over the interface entirely for the duration of a session.

## Decision

Ship shell functions (not scripts, not daemons) that the user pastes
into `~/.bashrc`/`~/.zshrc`:

1. `sudo service network-manager stop` and `sudo service wicd stop` —
   disable the competitors;
2. `sudo killall wpa_supplicant` — clear any stale supplicant;
3. `sudo wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf -B` —
   associate and authenticate;
4. `sudo dhclient -r wlan0` then `sudo dhclient wlan0` — release the old
   lease and request a fresh one;
5. a separate `refresh()` that only re-runs `dhclient wlan0`.

Provide duplicate `*1` variants hard-coded to `wlan1` for machines whose
interface enumerates differently, chosen by checking `iwconfig`.

## Consequences

- Deterministic, observable connection: every step runs in the user's
  terminal, so failures are visible instead of buried in a daemon.
- Hard-coded interface names and service names made the helpers brittle
  across machines and distros — hence the duplicated `wlan1` pair rather
  than a parameter (`internet wlan1`), a simplification the author chose
  for copy-paste ergonomics.
- The one bug in repo history lived here: `refresh()` originally ran
   `dhclient -r` alone, which *releases* the lease and disconnects the
   interface; fixed in `b6293c0`.
- Killing NetworkManager breaks non-Wi-Fi managed networking until the
   user restarts it — acceptable in 2013, surprising today.

## Revisit

Commit `e9c0557` invited a startup script to automate this; none was
ever contributed. Modern NetworkManager handles EAP-TTLS natively, which
is why the repo is archived rather than evolved.
