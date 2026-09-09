# AGENTS.md — working on this repository

Guidance for coding agents and automation touching this repo.

## What this repo is

A two-file, archived (2013-era) project: a `wpa_supplicant.conf` template
for joining UIUC's IllinoisNet WPA2-Enterprise Wi-Fi from Linux
(EAP-TTLS + MSCHAPv2), plus README shell helpers that stop
network-manager/wicd and run `wpa_supplicant` + `dhclient` manually.
There is no build system, no package manifest, no CI, and no test suite.

## Commands

```bash
git log --oneline                 # inspect history (11 commits + docs pass)
git show <sha>                    # inspect a commit
git status --porcelain            # verify clean tree
```

There is nothing to build, lint, or test. Verification is entirely
inspection-based (see "Verifying changes" below).

**Never execute** `internet`, `internet1`, `refresh`, `refresh1`,
`wpa_supplicant`, `dhclient`, `service network-manager stop`, or any
other connectivity/login command from this repo — they mutate the host's
networking. The scripts are documentation, not code to run.

## Architecture map

```text
wpa_supplicant.conf   # the only "source file": IllinoisNet EAP block + example PSK block
README.md             # the only "program": user follows prose + shell snippets
CHANGELOG.md          # history, newest first
architectural-diary/  # narrative + ADRs
prompt.md             # one-shot recreation prompt
```

Data flow (documented, not coded): user copies `wpa_supplicant.conf` to
`/etc/wpa_supplicant.conf`, substitutes their NetID into `identity` and
their password or NT hash into `password`, then the README's `internet()`
function stops competing daemons → starts `wpa_supplicant` on `wlan0`/
`wlan1` → obtains a DHCP lease via `dhclient`.

## Conventions

- Commit messages: Conventional Commits (`type: imperative subject`,
  subject ≤72 chars, body explains the why). All 11 original commits
  were rewritten to this format on 2026-09-08 (see CHANGELOG.md).
- Markdown for all documentation; fenced ```bash blocks for shell
  snippets.
- The wpa_supplicant dialect is `network={ ... }` blocks with fields
  `ssid`, `key_mgmt`, `eap`, `pairwise`, `phase2`, `identity`,
  `password`, `psk`.

## Gotchas

1. **Credentials are the top risk.** `wpa_supplicant.conf` must contain
   only placeholders (`netid`, `password`, dummy `psk`). Never commit a
   real NetID, password, NT hash, or PSK — and never write one into any
   doc. The README's `wpa_passphrase NetworkName NetworkPassword` form
   leaks passphrases into shell history; document the interactive form
   instead.
2. **`dhclient -r` releases; it does not renew.** The original README had
   this bug in `refresh()`; fixed in commit `b6293c0`. Don't reintroduce.
3. **README snippets are 2013-era.** `service network-manager stop`,
   `wicd`, `iwconfig`, `dhclient -r wlan0` predate systemd, `iwd`, and
   modern NetworkManager. Keep them as historical content (clearly
   framed as legacy); do not "modernize" them in place without adding a
   current-distro alternative.
4. **Case-only filename variants** (`readme.md`, `Readme.md`) behave
   strangely on macOS's case-insensitive filesystem — the canonical name
   is `README.md`.
5. **History was rewritten once** (2026-09-08, messages-only,
   `--force-with-lease`). Pre-rewrite refs live on local branch
   `backup/pre-docs-20260908` only; never push that branch. If a remote
   still has old SHAs, do not "merge" them — reclone or force-push with
   lease from this repo.
6. No `.gitignore` exists; untracked scratch dirs appear in status by
   design. Check `git status --porcelain` before committing.

## Verifying changes

- Docs-only edits: proofread in a markdown renderer; confirm no real
  credentials or live hostnames/URLs beyond the public GitHub credit
  link; confirm shell snippets are inside fenced code blocks (so nothing
  executes when rendered).
- `wpa_supplicant.conf` edits: `wpa_supplicant` syntax check only if the
  binary exists and can run in check mode (`wpa_cli`/`wpa_supplicant -c
  file -i dummy` is NOT safe to run — it requires privileges and touches
  networking; prefer eyeball verification of field names against the
  wpa_supplicant.conf(5) man page).
- Always: `git status --porcelain` is clean before you commit; `git log
  --oneline` and `git diff --stat HEAD~1` reflect exactly what you
  intended.

## Pointers

- History and per-commit detail: [CHANGELOG.md](CHANGELOG.md)
- Design narrative and decisions: [architectural-diary/main.md](architectural-diary/main.md)
- One-shot recreation prompt: [prompt.md](prompt.md)
- User-facing instructions: [README.md](README.md)
