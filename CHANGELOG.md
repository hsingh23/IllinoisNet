# Changelog

All notable changes to this project. Newest first.

> **History rewrite note (2026-09-08):** the original commit messages
> ("Good", "Better readme" ×7, "Make it work", "Initial commit") were
> rewritten in place to conventional-commit format via a messages-only
> `git filter-branch`. File contents, trees, authors, and dates are
> byte-for-byte unchanged; only the messages differ. Commit SHAs changed
> as a side effect (old tip `eabe205` → new tip `80771c1`). A pre-rewrite
> snapshot exists locally as branch `backup/pre-docs-20260908`.

## 2026-09-08 — documentation pass

- Added `README.md` (modern rewrite, legacy instructions preserved), `AGENTS.md`,
  this `CHANGELOG.md`, `architectural-diary/`, and `prompt.md` (one-shot
  recreation prompt).
- Rewrote all 11 historical commit messages to conventional-commit format
  (messages-only history rewrite, force-pushed with lease).

## 2016-02-28 — `80771c1`

- **docs: fix "network-manger" typo in README helper functions**
- Corrects the service name in the `internet()` and `internet1()` shell
  snippets so copy-pasted `service network-manager stop` commands work.

## 2013-09-09 — `e0103ce`

- **docs: reformat README code block and credit Zack**
- Dedents the shell functions inside the fenced bash block, adds a
  thank-you to [zach297](https://github.com/zach297) for demonstrating the
  setup, and adds a trailing newline at end of file.

## 2013-09-07 — `e9c0557`

- **docs: invite startup-script contributions in README**
- Replaces the closing "make this run on boot" tip with "I welcome a
  startup script" to invite contributions.

## 2013-09-07 — `86440c7`

- **docs: add wlan1 variants of internet/refresh functions**
- Adds `internet1()`/`refresh1()` helpers for machines whose wireless
  interface is `wlan1` instead of `wlan0`, and reorders the setup intro
  so the copy-config step precedes the credential-fill step.

## 2013-09-07 — `f433744`

- **docs: collapse README title into a single heading**
- Replaces the duplicated "IllinoisNet" / "#Get IllinoisNet working on
  linux!!" header lines with one descriptive heading.

## 2013-09-07 — `8e37e4a`

- **docs: merge setup sub-headings into one bolded paragraph**
- Combines the "copy the included wpa_supplicant" and "fill in your
  password and netid" headings into a single bolded instruction paragraph.

## 2013-09-07 — `ca3667f`

- **docs: restructure README with headings and fenced code blocks**
- Promotes title/setup lines to markdown headings, rewrites the
  password-hash instructions, adds a plaintext-password permissions
  warning, adds the "To add other networks" (`wpa_passphrase`) section,
  and wraps snippets in fenced bash blocks.

## 2013-09-07 — `b6293c0`

- **fix: drop -r flag from refresh() so it renews the DHCP lease**
- `sudo dhclient -r wlan0` only *releases* the lease and disconnects the
  interface; `sudo dhclient wlan0` actually requests a lease, so the
  documented `refresh()` helper is fixed to use the latter.

## 2013-09-07 — `bc21664`

- **docs: split dense README into paragraphs and indent snippets**
- Breaks the wall-of-text README into paragraphs, indents the example
  network block and shell functions, and adds markdown hard line breaks.

## 2013-09-07 — `88d6ce0`

- **feat: add wpa_supplicant.conf and full setup instructions**
- Adds the core artifact: a `wpa_supplicant.conf` with an IllinoisNet
  WPA-EAP/TTLS/MSCHAPv2 network block (placeholder credentials) plus an
  example PSK network. Documents the full setup in the README: copying
  the config to `/etc`, generating the MSCHAPv2 password hash, adding
  networks via `wpa_passphrase`, and `internet()`/`refresh()` shell
  helpers that stop network-manager/wicd and bring up wpa_supplicant
  with dhclient.

## 2013-09-07 — `7714f19`

- **chore: initialize repository with README stub**
- First commit: a four-line README stating the project's purpose —
  getting IllinoisNet working on Linux.
