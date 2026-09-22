<p align="center">
  <img src="kernel-panic-256.png" width="128" height="128" alt="Kernel Panic logo">
</p>

<h1 align="center">KERNEL PANIC</h1>
<p align="center"><b>A neon twin-stick roguelite for Linux, with encrypted peer-to-peer co-op.</b></p>

<p align="center">
  <img alt="C++17" src="https://img.shields.io/badge/C%2B%2B-17-00599C?style=flat-square&logo=cplusplus">
  <img alt="SDL2" src="https://img.shields.io/badge/SDL-2-1a1a2e?style=flat-square">
  <img alt="libsodium" src="https://img.shields.io/badge/crypto-libsodium-6a5acd?style=flat-square">
  <img alt="Platform" src="https://img.shields.io/badge/platform-Linux-orange?style=flat-square&logo=linux">
  <img alt="No assets" src="https://img.shields.io/badge/assets-zero-ff5fa2?style=flat-square">
</p>

<p align="center">
  <img src="screenshots/gameplay.png" width="90%" alt="Boss fight against Botnet Hydra">
</p>

Malware is flooding `/boot`. You're the last daemon standing. Dash, shoot, and level up through escalating waves, pick from 20 upgrades and two kill-charged special powers, and take down three bosses with distinct attack patterns — solo, or with up to three friends, hosted with nothing but a single invite code you paste and share.

**Everything you see is generated at runtime.** No sprite sheets, no sample library — every ship, enemy, boss, UI element and soundtrack note is drawn or synthesized by ~2,000 lines of C++ the moment you launch the binary. Nothing to download beyond the source.

<div align="center">

**20** upgrades · **2** active powers · **3** bosses · **5** ships · **4**-player P2P co-op · **0** asset files

</div>

## Contents

- [Screenshots](#screenshots)
- [Features](#features)
- [Install](#install)
- [Multiplayer](#multiplayer)
- [Controls](#controls)
- [Outside the game](#outside-the-game)
- [Options](#options)
- [How it works](#how-it-works)
- [Troubleshooting](#troubleshooting)
- [Known limits](#known-limits)
- [License](#license)

## Screenshots

<table>
<tr>
<td><img src="screenshots/title.png" width="100%" alt="Title screen"></td>
<td><img src="screenshots/shop.png" width="100%" alt="Ship shop"></td>
</tr>
<tr>
<td align="center"><sub>Title screen</sub></td>
<td align="center"><sub>Ship shop — spend coins earned from playing</sub></td>
</tr>
<tr>
<td><img src="screenshots/coop.png" width="100%" alt="Two-player co-op"></td>
<td><img src="screenshots/gameplay.png" width="100%" alt="Boss fight"></td>
</tr>
<tr>
<td align="center"><sub>Co-op — every player's ship, HP and level shown live</sub></td>
<td align="center"><sub>Wave 6, Botnet Hydra boss fight</sub></td>
</tr>
</table>

## Features

- **Twin-stick roguelite core** — dash, aim-and-shoot, level up mid-run, pick 1 of 3 upgrades each time from a pool of 20: piercing rounds, homing shots, orbiting blades, lightning chains, shockwaves, lifesteal, dodge chance, splash damage, and more.
- **Two active powers, independent of your upgrades** — **Firewall** (`Q`), a personal shield on its own cooldown, and **Turbo** (`R`), a team-wide kill-charged buff separate from the sudo ultimate.
- **Three bosses, each with multiple phases** — Botnet Hydra, Zero-Day Exploit, Segfault — with distinct bullet patterns that escalate as their health drops.
- **Ships and coins** — every run earns coins from your score and kills, kept permanently. Spend them in the shop on 5 ships with different HP, speed, damage and size trade-offs.
- **Peer-to-peer co-op, up to 4 players, no server to run or pay for** — host a game, share one invite code, done. Details in [Multiplayer](#multiplayer).
- **Desktop integration** — desktop notifications, a live JSON status file, and a hook system so external scripts can react to waves, bosses, and more. Details in [Outside the game](#outside-the-game).

## Install

Needs a C++17 compiler, SDL2, and libsodium. `libnotify` and `miniupnpc` are optional — desktop notifications and automatic router port-opening — the game runs fine without either.

```bash
# Gentoo
sudo emerge --ask media-libs/libsdl2 dev-libs/libsodium x11-libs/libnotify net-libs/miniupnpc
# Arch / Manjaro
sudo pacman -S --needed sdl2 libsodium libnotify miniupnpc base-devel
# Debian / Ubuntu / Mint / Pop!_OS
sudo apt install build-essential libsdl2-dev libsodium-dev libnotify-bin libminiupnpc-dev
# Fedora / Nobara
sudo dnf install gcc-c++ SDL2-devel libsodium-devel libnotify miniupnpc-devel
# openSUSE
sudo zypper install gcc-c++ libSDL2-devel libsodium-devel libnotify-tools miniupnpc-devel
```

Then clone and build:

```bash
git clone https://github.com/<your-username>/kernel-panic.git
cd kernel-panic
make            # builds ./kernel-panic
make test       # optional: runs the network/security test suite under AddressSanitizer + UBSan
make install    # installs to ~/.local: binary, app-menu entry with icon, default hooks
```

`kernel-panic` should now be in your app menu (right-click it for "Host a co-op game"), or run it directly:

```bash
~/.local/bin/kernel-panic
```

If it isn't found by name in a terminal, add `~/.local/bin` to your `PATH`:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

## Multiplayer

1. **Host:** title screen → **HOST CO-OP**. The lobby shows one invite code starting with `KP-`. Press **COPY INVITE** and send it.
2. **Friends:** **JOIN CO-OP**, paste the code (`Ctrl+V`), press Enter.
3. **Host:** press **START GAME** once everyone's in.

No server, no account, and no port forwarding in the common case. The invite carries the host's IPv6 address, a UPnP-opened port, and/or a STUN-discovered public address; the joining game tries all of them and uses whichever answers first. If none work, use a VPN like Tailscale, the same LAN, or forward the UDP port shown in the lobby by hand.

Every session gets a fresh random secret. The handshake is an ephemeral X25519 key exchange (forward secrecy); every packet after that is authenticated and encrypted with XChaCha20-Poly1305. Unauthenticated traffic gets no reply and creates no state on the host. `make test` runs the suite that checks this — tampering, replay, 60,000 garbage packets, simulated packet loss, and typo detection in the invite code.

The invite code is effectively a password plus your address: only share it with people you trust, and note it changes every time you host. The host is authoritative and trusted by its clients, same as in any other co-op game.

## Controls

| Action | Key |
|---|---|
| Move | `WASD` |
| Aim / fire | Mouse, hold to fire |
| Dash | `Space` |
| Firewall (shield) | `Q` |
| Sudo (ultimate) | `E` |
| Turbo (team buff) | `R` |
| Menu / pause | `P` or `Esc` |
| Fullscreen | `F11` |
| Screenshot | `F12` |

Gamepad: left stick move, right stick aim + fire, `A` dash, `B`/`X` sudo, `Y` firewall, D-pad up turbo, `Start` menu. Level-up picks: click, `1`/`2`/`3`, or arrows + Enter.

## Outside the game

On events (start, wave, boss, boss defeated, sudo, level-up, game over, new high score) the game:

1. sends a desktop notification and flashes the taskbar if the window isn't focused,
2. writes `~/.cache/kernel-panic/live.json` (current wave, score, HP, color) for scripts and widgets,
3. runs `~/.config/kernel-panic/hooks/on_<event>` and `on_any`, with `KP_*` environment variables.

The default `on_any` hook recolors your [kitty](https://sw.kovidgoyal.net/kitty/) terminal to match the current wave. See `hooks/examples/` for a Mullvad-reconnect-on-boss hook, an OpenRGB hook, and a per-wave wallpaper hook. Best-score fastfetch snippet:

```jsonc
{ "type": "command", "key": "| Kernel Panic", "text": "cat ~/.local/share/kernel-panic/best.txt" }
```

## Options

```
--windowed / --fullscreen   --no-bloom   --no-notify   --no-hooks   --no-vsync
--host [--port N]           host a co-op game directly
--join KP-INVITE            join a co-op game with a pasted invite
```

Settings persist in `~/.config/kernel-panic/config.ini`.

## How it works

<details>
<summary>No asset files — how does that work?</summary>

<br>

Every ship, enemy and boss is a small hand-written triangle/polygon mesh, shaded and glowed at draw time. Text is a baked-in 5×7 bitmap font. The soundtrack and every sound effect are additive synthesis — sine/square/saw/triangle oscillators mixed and enveloped in C++, no samples anywhere. It keeps the binary small and the whole game reviewable as plain source.

</details>

<details>
<summary>How does the co-op stay secure with no server?</summary>

<br>

The invite code is a Crockford-base32 string encoding a random 128-bit room secret plus up to four of the host's candidate network addresses. A joining client dials all of them at once. The handshake does an X25519 key exchange authenticated with a keyed hash of the room secret (Noise-style), then every subsequent packet is sealed with XChaCha20-Poly1305 using per-direction keys and a replay window. A packet that fails authentication gets silently dropped — the host allocates nothing for it. See `net.h` for the implementation and `tests/nettest.cpp` for the test suite.

</details>

## Troubleshooting

<details>
<summary>App-menu icon doesn't launch anything</summary>

<br>

`make install` writes an absolute path into the launcher, so a fresh `make install` should fix this. Some menus cache entries and need a log out/in to notice a new one.

</details>

<details>
<summary>Crashes on launch</summary>

<br>

Run `~/.local/bin/kernel-panic` from a terminal to see the actual error. Try forcing a video backend: `SDL_VIDEODRIVER=x11 kernel-panic` or `SDL_VIDEODRIVER=wayland kernel-panic`.

</details>

<details>
<summary>No sound</summary>

<br>

Try forcing an audio backend: `SDL_AUDIODRIVER=pipewire kernel-panic` or `SDL_AUDIODRIVER=pulseaudio kernel-panic`.

</details>

## Known limits

No client-side movement prediction yet — co-op movement feels delayed on a high-latency link. Up to 4 players; nobody can join mid-run. Tested on loopback with simulated packet loss, not over real, varied internet paths.

## License

No license has been chosen yet for this repository — add a `LICENSE` file before treating this as open source in the legal sense (MIT is a common, permissive default for a project like this).
