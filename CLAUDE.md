# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A ZMK user-config repo for the **Eyelash Peripherals Corne** — a wireless split keyboard that is *not* foostan's Corne and will not work with standard `corne` firmware. Each half is its own board (`eyelash_corne_left` / `eyelash_corne_right`), with a nice!view display, a 5-way joystick in the center column and a backlight. The physical keyboard has **no encoder**, even though the config still enables EC11 support. ZMK is pinned to `v0.3.0` in `config/west.yml`.

## Building

There is no local toolchain set up (`zmk/`, `modules/`, `.west/`, `build/` are gitignored). Firmware is built by GitHub Actions:

- `.github/workflows/build.yml` — calls ZMK's `build-user-config.yml` on every push (except `keymap-drawer/**` changes) and produces the `.uf2` artifacts listed in `build.yaml`.
- `build.yaml` — the build matrix: right half, left half built with ZMK Studio (`studio-rpc-usb-uart`, locking disabled; artifact `eyelash_corne_studio_left`), and `settings_reset` for `nice_nano_v2`. The left half is the central.
- `.github/workflows/draw.yml` — on changes under `config/` or to `keymap_drawer.config.yaml`, regenerates `keymap-drawer/eyelash_corne.{yaml,svg}` and **pushes a `[Draw] <msg>` commit back**. Expect this commit after every keymap push; pull before continuing work.

Locally, the only practical check is reading the keymap carefully; changes can't be compiled without a `west` workspace.

## Where things live

- `config/eyelash_corne.keymap` — **the keymap actually built.** Behaviors, combos, encoder bindings and the four layers (0 QWERTY, 1 NUMBER, 2 SYMBOL, 3 Fn).
- `config/eyelash_corne.conf` — Kconfig (sleep, RGB, NKRO, pointing, debounce, soft-off, TX power).
- `config/eyelash_corne.json` — physical layout for keymap-drawer / ZMK Studio.
- `config/west.yml` — ZMK plus the vendor module `eyelash_corne` (`github.com/a741725193/zmk-new_corne`, `revision: main`). Upstream `main` has since (June 2026) dropped `boards/arm/eyelash_corne` and become a `nice_nano_v2` + `eyelash_corne_left/right` **shield**, so it no longer provides the boards `build.yaml` asks for.
- `boards/arm/eyelash_corne/` — the board definition this repo actually builds against: a vendored copy of upstream `d7fa543` (May 2025). `zephyr/module.yml` sets `board_root: .` so the repo is a Zephyr module and the board is found here. **Don't delete it** while `build.yaml` uses `board: eyelash_corne_*`.
- `keymap_drawer.config.yaml` — drawing style and `raw_binding_map` legends. Legends are keyed by the exact binding string (e.g. `"&hm LEFT_SHIFT F"`), so they silently fall back to raw text when a binding in the keymap changes.
- `keymap-drawer/` — generated output; don't edit by hand.

## Key positions

48 positions, numbered row by row across both halves (needed for `combos` `key-positions`):

```
 0  1  2  3  4  5 |      6      |  7  8  9 10 11 12
13 14 15 16 17 18 | 19  20  21  | 22 23 24 25 26 27
28 29 30 31 32 33 | 34      35  | 36 37 38 39 40 41
         42 43 44 |             | 45 46 47
```

Center column: 6/19/20/21/35 are the joystick (up/left/press/right/down). 34 is the encoder-press slot, and since no encoder is fitted, bindings there do nothing.

## Keymap conventions

- Home-row mods use the custom `hm` hold-tap on **S D F G / H J K L** (GUI, ALT, CTRL, SHIFT mirrored). This placement is intentional; don't propose moving it to A S D F.
- Bootloader and reset are not on any layer: they are combos restricted to the Fn layer (`layers = <3>`) — bootloader on 0 1 2 / 10 11 12, reset on 0 13 28 / 12 27 41. Soft-off is the Q S Z combo (1 15 29) held for 2 s; wake with the reset switch.
- When changing a binding that has a custom legend, update the matching key in `keymap_drawer.config.yaml` too.
