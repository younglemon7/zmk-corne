# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A ZMK user-config repo for the **Eyelash Peripherals Corne** — a wireless split keyboard that is *not* foostan's Corne and will not work with standard `corne` firmware. Each half is its own board (`eyelash_corne_left` / `eyelash_corne_right`), with a nice!view display, a 5-way joystick in the center column and backlight LEDs. The physical keyboard has **no encoder** and no switch in the encoder slot; EC11 support is disabled (`left_encoder` stays `disabled` as defined in the board dtsi). ZMK is pinned to `v0.3.0` in `config/west.yml`.

## Building

There is no local toolchain set up (`zmk/`, `modules/`, `.west/`, `build/` are gitignored). Firmware is built by GitHub Actions:

- `.github/workflows/build.yml` — calls ZMK's `build-user-config.yml@v0.3.0` on push to `main` only (except `keymap-drawer/**` changes) and produces the `.uf2` artifacts listed in `build.yaml`. Feature branches are not built automatically; build one with `gh workflow run build.yml --ref <branch>`.
- `build.yaml` — the build matrix: right half, left half built with ZMK Studio (`studio-rpc-usb-uart`, locking disabled; artifact `eyelash_corne_studio_left`), and `settings_reset` for `nice_nano_v2`. The left half is the central.
- `.github/workflows/draw.yml` — on push to `main` touching `config/` or `keymap_drawer.config.yaml`, regenerates `keymap-drawer/eyelash_corne.{yaml,svg}` and **pushes a `[Draw] <msg>` commit back to `main`**. Pull `main` after merging a keymap change.

Locally, the only practical check is reading the keymap carefully; changes can't be compiled without a `west` workspace.

## Where things live

- `config/eyelash_corne.keymap` — **the keymap actually built.** Behaviors, combos and the four layers (0 QWERTY, 1 NUMBER, 2 SYMBOL, 3 Fn).
- `config/eyelash_corne.conf` — Kconfig (sleep after 10 min idle, NKRO, debounce, soft-off, TX power). RGB underglow, backlight and mouse/pointing are intentionally disabled.
- `config/eyelash_corne.json` — physical layout for keymap-drawer / ZMK Studio.
- `config/west.yml` — only ZMK `v0.3.0`. The vendor repo (`github.com/a741725193/zmk-new_corne`) was removed from the manifest: since June 2026 its `main` is a `nice_nano_v2` + `eyelash_corne_left/right` **shield** on a ZMK fork and no longer provides the boards `build.yaml` asks for.
- `boards/arm/eyelash_corne/` — the board definition this repo actually builds against: a vendored copy of upstream `d7fa543` (May 2025). `zephyr/module.yml` sets `board_root: .` so the repo is a Zephyr module and the board is found here. **Don't delete it** while `build.yaml` uses `board: eyelash_corne_*`.
- `keymap_drawer.config.yaml` — drawing style and `raw_binding_map` legends. Legends are keyed by the exact binding string (e.g. `"&hml LEFT_SHIFT G"`), so they silently fall back to raw text when a binding in the keymap changes.
- `keymap-drawer/` — generated output; don't edit by hand.

## Key positions

48 positions, numbered row by row across both halves (needed for `combos` `key-positions`):

```
 0  1  2  3  4  5 |      6      |  7  8  9 10 11 12
13 14 15 16 17 18 | 19  20  21  | 22 23 24 25 26 27
28 29 30 31 32 33 | 34      35  | 36 37 38 39 40 41
         42 43 44 |             | 45 46 47
```

Center column: 6/19/20/21/35 are the joystick (up/left/press/right/down). 34 is the encoder-press slot; nothing is fitted there, so it is `&none` on every layer (`&trans` on SYMBOL).

## Keymap conventions

- Home-row mods are on **S D F G / H J K L** (GUI, ALT, CTRL, SHIFT mirrored). This placement is intentional; don't propose moving it to A S D F. They are "timeless" hold-taps: `hml` (left) and `hmr` (right), `balanced`, 280/175/150 ms, with `hold-trigger-key-positions` listing the opposite hand + thumbs (the joystick counts as right hand) and `hold-trigger-on-release`. Same-hand mod+key (e.g. Cmd+C via S) does not work by design; use the `LGUI` thumb or the other hand. Keep both position lists in sync if keys move.
- Bootloader and reset are not on any layer: they are combos restricted to the Fn layer (`layers = <3>`) — bootloader on 0 1 2 / 10 11 12, reset on 0 13 28 / 12 27 41. Soft-off is the Q S Z combo (1 15 29), also Fn-only, held for 2 s; wake with the reset switch. No combo is active on the base layer.
- Bluetooth/output keys live only on the Fn layer, left half (hold Space, press with the same hand): BT profiles 0–3 on Z X C V (29–32), `OUT_TOG` on B (33), `BT_CLR` on the leftmost middle key (13). 13 is also part of the reset combo `0 13 28`, so `BT_CLR` fires only after the combo timeout. There is no `BT_CLR_ALL` key; flash `settings_reset` for a full wipe.
- Fn + G (18) is `&ext_power EP_ON`: re-enables the external power rail (nice!view displays) on both halves. Needed because the old RGB-underglow firmware could leave a half with ext power saved as off.
- System keys on Fn, right half: screenshots H (22) = Cmd+Shift+4 (area to file), N (36) = Cmd+Ctrl+Shift+4 (area to clipboard); media J K L (23–25) = previous / play-pause / next; volume M , . (37–39) = mute / down / up.
- `README.md` (in Russian) has a user-facing "Управление" table of all Fn combos and BT keys; update it whenever combos or Fn-layer control keys move.
- When changing a binding that has a custom legend, update the matching key in `keymap_drawer.config.yaml` too.
