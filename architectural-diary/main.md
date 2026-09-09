# Architectural diary — IllinoisNet (Linux)

Narrative history of this repository and index of its decision records.
Commit hashes are the post-2026-rewrite SHAs (see CHANGELOG.md for the
rewrite note).

## The project in one paragraph

In September 2013, connecting Linux laptops to UIUC's IllinoisNet
WPA2-Enterprise Wi-Fi was unreliable with the desktop network managers
of the day (NetworkManager, wicd). This repo captures a manual,
deterministic workaround: a two-block `wpa_supplicant.conf` (IllinoisNet
via EAP-TTLS/MSCHAPv2, plus an example PSK network) and a set of copy-
into-your-shellrc functions that stop the competing daemons, start
`wpa_supplicant` against `/etc/wpa_supplicant.conf`, and pull a DHCP
lease with `dhclient`. There is no executable code in the repo — the
configuration file and the README *are* the product.

## Timeline

| Date | Commit | What happened |
| --- | --- | --- |
| 2013-09-07 | `7714f19` | Repo initialized with a 4-line README stub stating the goal. |
| 2013-09-07 | `88d6ce0` | The substance arrives: `wpa_supplicant.conf` + full README instructions (copy to /etc, NT-hash generation, `wpa_passphrase`, `internet()`/`refresh()` helpers). |
| 2013-09-07 | `bc21664` | Readability pass: paragraphs, indentation. |
| 2013-09-07 | `b6293c0` | **Only functional fix in repo history:** `refresh()` used `dhclient -r`, which *releases* a lease (disconnecting the user) instead of requesting one; flag dropped. |
| 2013-09-07 | `ca3667f` | Big README restructure: headings, fenced code blocks, plaintext-password permissions warning, "To add other networks" section. |
| 2013-09-07 | `8e37e4a` | Setup sub-headings merged into one bolded paragraph. |
| 2013-09-07 | `f433744` | Duplicated title lines collapsed into one heading. |
| 2013-09-07 | `86440c7` | `wlan1` variants (`internet1`/`refresh1`) added; setup steps reordered to execution order. |
| 2013-09-07 | `e9c0557` | Closing line changed to "I welcome a startup script" — an unfulfilled invitation; the boot-time integration was never built. |
| 2013-09-09 | `e0103ce` | Final polish: dedented code block, trailing newline, credit to zach297 who demonstrated the procedure. |
| 2016-02-28 | `80771c1` | Typo fix (`network-manger` → `network-manager`) — the repo's last functional touch for 13 years; proof people were still copy-pasting the snippets. |
| 2026-09-08 | (docs) | Documentation pass: README rewrite, AGENTS.md, CHANGELOG.md, this diary, `prompt.md`; all 11 commit messages rewritten to Conventional Commits via messages-only `filter-branch` + `--force-with-lease`. |

## Shape of the system

```text
Repo (all static, no build, no deps)
├── wpa_supplicant.conf      ← sole "source": 2 network blocks, placeholder creds
└── README.md                ← sole "program": prose + shell functions the user copies

Runtime (on the user's machine, 2013 Debian-era Linux)
/etc/wpa_supplicant.conf     ← user-edited copy with real NetID + password/hash
internet()/internet1()       ← stop network-manager & wicd → kill wpa_supplicant
                               → wpa_supplicant -i wlanN -B → dhclient
refresh()/refresh1()         ← re-run dhclient to renew a lease
```

Key structural facts:

- **Everything is a template.** The committed config holds placeholders
  (`identity="netid"`, `password="password"`, a dummy `psk`); the user
  materializes real credentials into `/etc`, never into git.
- **The repo optimizes for copy-paste.** Success metric: a user with a
  campus NetID reaches the internet in one `internet` invocation. Nine of
  eleven commits are README refinement toward that.
- **No abstractions, deliberately.** No installer, no boot script, no
  distro detection — the author explicitly invited a startup script
  (commit `e9c0557`) and never received/built one.

## Decision records

- [0001 — Use EAP-TTLS with MSCHAPv2 phase 2 for IllinoisNet](decisions/0001-eap-ttls-mschapv2-for-illinoisnet.md)
- [0002 — Store an NT password hash, not cleartext, in the config](decisions/0002-nt-password-hash-not-cleartext.md)
- [0003 — Bypass NetworkManager/wicd with manual wpa_supplicant + dhclient helpers](decisions/0003-manual-wpa-supplicant-over-network-manager.md)
- [0004 — Ship a template config + README instead of a script](decisions/0004-template-and-readme-not-a-script.md)
- [0005 — Commit only placeholder credentials](decisions/0005-placeholder-credentials-only.md)

## Retrospective notes

- The single real bug (`dhclient -r` in `refresh()`) is a neat example of
  a docs-delivered bug: the code lived in a README, so the "fix" is a
  markdown edit and users only get it by re-copying the function.
- The 2016 typo fix suggests the audience kept finding the repo via
  search years after the author left campus — the reason it's worth
  archiving clearly rather than deleting.
- The 2026 docs pass deliberately preserved every legacy command verbatim
  (inside fenced blocks, marked legacy) instead of rewriting history in
  place; the history *messages* were rewritten, but the trees are
  byte-identical to 2016.
