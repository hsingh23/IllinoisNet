# ADR 0002 — Store an NT password hash, not cleartext, in the config

- Date: 2013-09-07 (commit `88d6ce0`, warning formalized in `ca3667f`)
- Status: accepted

## Context

`wpa_supplicant.conf` must end up on disk at `/etc/wpa_supplicant.conf`
containing something MSCHAPv2 can use: either the campus password in
cleartext, or its NT hash (MD4 over the UTF-16LE encoding). The file is
root-owned in `/etc` but users commonly keep editable copies; a cleartext
campus password on disk is a credential-exposure risk (single sign-on
for email, portal, etc.).

## Decision

Document generating the NT hash on the user's machine and storing it:

```bash
echo -n 'YOUR_PASSWORD_HERE' | iconv -t utf16le | openssl md4
```

then setting `password=hash:<generated hash>` in the network block.
Support cleartext as a fallback for convenience, but pair it with an
explicit warning to restrict file permissions (`chmod 600`).

## Consequences

- The cleartext campus password need not persist on disk; only the MD4
  hash does, which cannot be reversed offline by reading the file.
- The hash is still a *usable authenticator* for MSCHAPv2 (pass-the-hash
  works against the campus RADIUS), so the file still requires tight
  permissions — the README says so, and AGENTS.md repeats it.
- The `echo -n 'password'` command itself puts the password in shell
  history momentarily; acceptable trade-off documented as legacy.

## Revisit

MD4/NT hashing is obsolete cryptography by modern standards; kept as a
historical record of the 2013 workflow.
