# IllinoisNet (Linux) — archived

Configuration and instructions for connecting a Linux machine to
**IllinoisNet**, the University of Illinois Urbana-Champaign (UIUC)
WPA2-Enterprise Wi-Fi network, circa 2013.

> **Status: archived / legacy.** This project was written in 2013 against
> Debian-era `wpa_supplicant`, `wicd`, `network-manager`, and `iwconfig`/
> `dhclient`. Modern distros (NetworkManager 1.x, `iwd`, `wpa_supplicant`
> with `ctrl_interface`) can usually join WPA2-Enterprise networks
> directly from their network applet, so most of the manual steps below
> are no longer necessary. The repo is kept for historical reference and
> as a worked example of a WPA-EAP/TTLS setup.
>
> Credit: the original procedure was shown to the author by
> [Zack (zach297)](https://github.com/zach297).

## What and why

In 2013 many Linux desktops struggled with UIUC's WPA2-Enterprise
(IllinoisNet) SSID: NetworkManager and wicd would fail or fight with a
manual `wpa_supplicant`. This repo packages the two things that made it
work reliably:

1. `wpa_supplicant.conf` — a **template** network block for IllinoisNet
   using `key_mgmt=WPA-EAP`, `eap=TTLS`, `phase2="auth=MSCHAPV2"`.
2. README shell helpers (`internet` / `refresh`) that stop the competing
   network managers, kill any running `wpa_supplicant`, start
   `wpa_supplicant` against the config, and obtain a DHCP lease.

## Features

- IllinoisNet EAP-TTLS/MSCHAPv2 `wpa_supplicant` network block (template —
  fill in your own credentials).
- Password hashing one-liner for MSCHAPv2 (`iconv` + `openssl md4`) so the
  cleartext password need not live in the config.
- `wlan0` and `wlan1` variants of the connect/refresh shell functions.
- Instructions for appending additional (PSK) networks via
  `wpa_passphrase`.

## Stack (2013-era)

- Linux with `wpa_supplicant` (WPA-EAP/TTLS support)
- `dhclient` (DHCP), `iwconfig` (interface inspection)
- `network-manager` and/or `wicd` (stopped during connect)
- `iconv` + `openssl` (NT-hash generation for MSCHAPv2)

## Quickstart (legacy instructions)

**Security first:** the config file ultimately holds your campus
credentials (NetID in `identity`, a password or password hash in
`password`). Fill in your own values, keep them out of git, and restrict
permissions — see the warning below. The committed file contains only
placeholders.

1. Copy the supplied config to `/etc/wpa_supplicant.conf`.
2. Fill in your own NetID (`identity`) and password in the IllinoisNet
   network block. To store an NT password hash instead of cleartext:

   ```bash
   echo -n 'YOUR_PASSWORD_HERE' | iconv -t utf16le | openssl md4
   ```

   and set `password=hash:YOUR_GENERATED_HASH_HERE`.

   A plaintext password also works but demands tight file permissions —
   other users on the machine (or over the network) must not be able to
   read the file:

   ```bash
   sudo chmod 600 /etc/wpa_supplicant.conf
   ```

3. Identify your wireless interface (`iwconfig` — typically `wlan0`;
   use the `*1` function variants if it is `wlan1`), then add the
   matching helpers to your `~/.bashrc` or `~/.zshrc`:

   ```bash
   function internet(){
     sudo service network-manager stop
     sudo service wicd stop
     sudo killall wpa_supplicant
     sudo wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf -B # you may need wlan1 — check iwconfig
     sudo dhclient -r wlan0 # release old lease; change to wlan1 if needed
     echo "refreshing the dhcp client - this could take a while or it could not work"
     sudo dhclient wlan0
     sudo iwconfig
     echo "and we are good to go - happy surfing"
   }
   function refresh(){
     sudo dhclient wlan0
   }
   function internet1(){
     sudo service network-manager stop
     sudo service wicd stop
     sudo killall wpa_supplicant
     sudo wpa_supplicant -i wlan1 -c /etc/wpa_supplicant.conf -B
     sudo dhclient -r wlan1
     echo "refreshing the dhcp client - this could take a while or it could not work"
     sudo dhclient wlan1
     sudo iwconfig
     echo "and we are good to go - happy surfing"
   }
   function refresh1(){
     sudo dhclient wlan1
   }
   ```

4. Run `internet` (or `internet1`) to connect; run `refresh` to renew a
   DHCP lease.

### Adding other networks

Generate an additional (WPA-PSK) network block with:

```bash
wpa_passphrase NetworkName NetworkPassword
```

It produces something like:

```bash
network={
    ssid="OtherHomeNetwork"
    psk=<long generated string>
}
```

Append blocks like this to `/etc/wpa_supplicant.conf` as needed. Note
that `wpa_passphrase` takes the passphrase as an argument, so it can
leak into shell history — prefer the interactive prompt form
(`wpa_passphrase NetworkName`) on shared machines.

## Repository structure

```text
.
├── README.md             # this file (legacy instructions preserved)
├── CHANGELOG.md          # per-commit history, newest first
├── AGENTS.md             # agent/automation guide for this repo
├── prompt.md             # one-shot recreation prompt for this project
├── architectural-diary/  # design narrative and decision records
│   ├── main.md
│   └── decisions/
└── wpa_supplicant.conf   # IllinoisNet EAP-TTLS template (placeholders only)
```

## Environment / inputs

There are no environment variables or secret names — everything is a
user-supplied value inside `wpa_supplicant.conf`:

| Config field  | Meaning                                   |
| ------------- | ----------------------------------------- |
| `identity`    | Your campus NetID (username)              |
| `password`    | Your password, or `password=hash:<NT hash>` |
| `psk`         | Pre-shared key for other (PSK) networks   |

Never commit real values for these fields.

## See also

- [CHANGELOG.md](CHANGELOG.md) — full commit history.
- [AGENTS.md](AGENTS.md) — how to work on this repo safely.
- [architectural-diary/main.md](architectural-diary/main.md) — how and
  why the project evolved.
- [prompt.md](prompt.md) — prompt that recreates this project from
  scratch.
