# ADR 0001 — Use EAP-TTLS with MSCHAPv2 phase 2 for IllinoisNet

- Date: 2013-09-07 (commit `88d6ce0`)
- Status: accepted, historical (campus infrastructure decision, not ours to change)

## Context

IllinoisNet is a WPA2-Enterprise network: authenticating requires an
802.1X EAP method, and the campus RADIUS infrastructure dictates which
methods it will accept. The two candidate methods commonly supported for
eduroam-style campus networks are EAP-TTLS and EAP-PEAP; both tunnel an
inner (phase 2) authentication, and both were workable from
`wpa_supplicant` on Linux at UIUC in 2013. The inner method had to match
what the RADIUS server verifies against campus credentials (NetID +
password) — MSCHAPv2, which relies on the NT hash of the password.

## Decision

Configure the IllinoisNet block with:

```text
key_mgmt=WPA-EAP
eap=TTLS
pairwise=CCMP
phase2="auth=MSCHAPV2"
identity=<NetID>
password=<password or NT hash>
```

## Consequences

- TTLS needs only the password (or its NT hash) on the client — no
  client certificate provisioning, which kept the setup to a single file
  a student could edit.
- Because MSCHAPv2 uses the NT hash, the client can *precompute* and
  store `password=hash:<NT hash>` instead of cleartext (ADR 0002).
- `pairwise=CCMP` pins the WPA2 unicast cipher; correct for the era.
- Locked to whatever the campus RADIUS accepted in 2013 — if UIUC later
  changed methods, the config would silently stop authenticating.

## Revisit

Not revisited since 2013; treat as a point-in-time record.
