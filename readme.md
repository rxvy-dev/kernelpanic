<p align="center">
  <img src="kernel-panic-256.png" alt="Kernel Panic icon" width="160">
</p>

<h1 align="center">KERNEL PANIC</h1>

<p align="center">
  A neon twin-stick roguelite for Gentoo Linux with encrypted peer-to-peer co-op for up to 4 players.<br>
  C++17 · SDL2 · libsodium · no assets: vector graphics and live-synthesized audio.
</p>

---

## Features

- **Twin-stick roguelite**: survive escalating waves of malware, collect XP, and pick upgrades on every level-up.
- **Co-op for up to 4 players** with no server: the host shares one invite code, friends paste it, done.
- **Encrypted, authenticated networking**: X25519 handshake, XChaCha20-Poly1305 data, replay protection.
- **Zero assets**: everything is drawn as vectors and the audio is synthesized at runtime, so the whole game is one binary.
- **Keyboard + mouse or gamepad**, with rumble.
- **Scriptable events**: hooks fire on wave, boss, level-up, game over and more. Recolor your terminal, sync RGB lighting, play sounds, whatever you like.
- **Hardened build**: stack protector, PIE, full RELRO, `_FORTIFY_SOURCE`.

## Gameplay

You are a process fighting to keep the system alive.

| | |
|---|---|
| **Enemies** | Worm, Trojan, Spy, Ransomware, Charger, Bomb (plus elite variants) |
| **Bosses** | BOTNET HYDRA, ZERO-DAY EXPLOIT, SEGFAULT (the last uses telegraphed laser sweeps) |
| **Upgrades** | CPU (fire rate, multishot, damage, pierce, crits), WEAPON (homing, orbiting blades, chain lightning, shockwave), SYSTEM (max HP, regen, pickup range), MOBILITY (speed, dash cooldown), and a heal |
| **Special** | Fill the meter and hit **sudo** to wipe the screen |

## Requirements

- Gentoo Linux with a desktop session (X11 or Wayland)
- A C++17 compiler (`g++` by default), `make`, `pkg-config`
- [SDL2](https://www.libsdl.org/) and [libsodium](https://libsodium.org/)
- Optional: `notify-send` from `x11-libs/libnotify` (desktop notifications), [miniupnpc](https://miniupnp.tuxfamily.org/) (lets the game open a router port when hosting)

## Build and install

Install the dependencies (`net-libs/miniupnpc` is optional and lets the game open its own router port when hosting):

```sh
sudo emerge --ask media-libs/libsdl2 dev-libs/libsodium net-libs/miniupnpc x11-libs/libnotify
```

Unpack, build, and install:

```sh
mv ~/Downloads/kernel-panic-native.tar.gz ~/
cd ~ && tar xf kernel-panic-native.tar.gz && cd kernel-panic-native
make && make install
```

Run it:

```sh
~/.local/bin/kernel-panic
```

Or start **Kernel Panic** from your app menu.

Other make targets:

```sh
make test       # network/security test suite under AddressSanitizer + UBSan
make uninstall  # remove the installed files
```

`make install` uses `PREFIX=$HOME/.local` by default. Override it with `make install PREFIX=/usr/local`.
If `kernel-panic` isn't found in a terminal, add `~/.local/bin` to your `PATH`.

UPnP support is detected automatically; if miniupnpc isn't installed, the build simply leaves it out.

## Controls

| Action | Keyboard / mouse | Gamepad |
|---|---|---|
| Move | `W` `A` `S` `D` | Left stick |
| Aim and fire | Mouse (hold to fire) | Right stick |
| Dash | `Space` | `A` |
| Sudo | `E` | `B` / `X` |
| Menu | `P` / `Esc` | `Start` |
| Fullscreen / screenshot | `F11` / `F12` | |

Level-up: click, press `1` `2` `3`, or use arrows + `Enter`.
In co-op the game keeps running while the menu or level-up screen is open (pick within 14 seconds).

## Co-op

No server, no accounts. One code does it all.

1. **Host:** choose **HOST CO-OP**. The lobby shows a long invite starting with `KP-`. Press **COPY INVITE** and send it to your friends.
2. **Friends:** choose **JOIN CO-OP**, paste the invite (`Ctrl+V`), press `Enter`.
3. When everyone is in, the host presses **START GAME**.

From the command line:

```sh
kernel-panic --host
kernel-panic --join KP-XXXXX-XXXXX-...
```

The invite contains a fresh random room secret plus the host's reachable addresses (up to 4). The game discovers those addresses itself and the joining player tries all of them at once, using the first that answers:

- **IPv6**: direct, no router setup, works if both sides have IPv6.
- **Public IPv4** via **UPnP** (asks your router to forward the port; opt-in toggle on the title screen; removed when you leave) or **STUN** (learns your public address from a free public STUN server, which only sees your IP).
- **VPN / Tailscale** and **LAN** addresses.

**Troubleshooting:** if you see "no public IPv4 found", use IPv6, Tailscale/ZeroTier, the same LAN, or forward UDP port `47474` (or the port shown in the lobby) on your router by hand.
Some VPNs block incoming connections. Mullvad has no port forwarding, so turn it off while hosting or use Tailscale instead.

## Security model

- Every session gets a new 128-bit secret. Without it, packets fail a MAC check, and the host creates no state and sends no reply.
- **Handshake:** ephemeral X25519 key exchange (forward secrecy) bound to the secret with keyed BLAKE2b.
- **Data:** XChaCha20-Poly1305 per direction, authenticated headers, replay window.
- Handshake replies are exactly as large as requests, so the host can't be used for traffic amplification.
- Peers, pending handshakes, message queues and datagram sizes are all bounded, and every network message goes through a bounds-checked reader.
- The host is trusted by its clients (it runs the game). Clients can't cheat state: the host only accepts their input.

**Things to know:**

- The invite works like a password plus your address. Anyone who has it can join while the lobby is open, and it reveals your IP to whoever receives it. It changes every time you host.
- Anyone you play with learns your IP. That is unavoidable in peer-to-peer.
- No software can be promised bug-free. `make test` runs the transport tests (tampering, replay, 60k garbage packets, typo detection, packet loss) under sanitizers, and the game's message handlers were fuzzed for 400k+ iterations, which found and fixed two out-of-bounds bugs. Real-world networks, UPnP routers and STUN servers could not be tested in development.

## Hooks: connect the game to the rest of your desktop

On these events the game sends a desktop notification, flashes the taskbar if the window is unfocused, writes `~/.cache/kernel-panic/live.json`, and runs your hook scripts:

`start` · `wave` · `boss` · `bossdead` · `sudo` · `levelup` · `gameover` · `highscore`

Hooks are executables in `~/.config/kernel-panic/hooks/` named `on_<event>`. `on_any` runs for every event.
They receive these environment variables:

`KP_EVENT` `KP_WAVE` `KP_SCORE` `KP_KILLS` `KP_LEVEL` `KP_HP` `KP_TIME` `KP_BOSS` `KP_COLOR` `KP_BEST`

The default `on_any` recolors [kitty](https://sw.kovidgoyal.net/kitty/) to match the game's current color. To enable it, add this to `~/.config/kitty/kitty.conf` and restart kitty:

```
allow_remote_control socket-only
listen_on unix:/tmp/kitty
```

Ready-to-copy examples live in `~/.config/kernel-panic/hooks/examples/` after install (source in [`hooks/examples/`](hooks/examples)):

| Example | What it does |
|---|---|
| `on_wave.openrgb` | Syncs keyboard/mouse RGB to the wave color (needs `openrgb`) |
| `on_gameover.sound` | Plays a system sound when you die |
| `on_boss.mullvad` | Reconnects Mullvad to a new exit on boss fights |

To use one, copy it to `~/.config/kernel-panic/hooks/` under the plain event name (for example `on_wave`) and make sure it's executable.

Show your best score in fastfetch:

```json
{ "type": "command", "key": "| Kernel Panic", "text": "cat ~/.local/share/kernel-panic/best.txt" }
```

## Command-line options

```
--windowed  --fullscreen  --no-bloom  --no-notify  --no-hooks  --no-vsync
--host [--port N]  --join INVITE
```

Settings persist in `~/.config/kernel-panic/config.ini`. Run `kernel-panic --help` for the full list.
Environment variables such as `KP_HEADLESS`, `KP_HOST_AUTO`, `KP_JOIN`, `KP_FUZZ` and `KP_NET_LOSS` exist for automated testing only.

## Project layout

```
kernelpanic.cpp        game, rendering, audio, input, hooks
net.h                  header-only encrypted UDP transport (libsodium)
tests/nettest.cpp      transport and security tests (make test)
hooks/                 default on_any hook and examples
icons/                 app icons (PNG + SVG)
kernel-panic.desktop   app-menu entry (includes a "Host a co-op game" action)
install-hooks.sh       installs hooks without overwriting your own
Makefile               build / test / install / uninstall
```

## Known limitations

- No client-side movement prediction, so movement can feel delayed on high-latency links.
- Up to 4 players; nobody can join mid-run.
- Tested on loopback with simulated packet loss, not yet on real internet paths.
