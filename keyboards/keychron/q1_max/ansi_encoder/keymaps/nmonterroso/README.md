# nmonterroso keymap — Keychron Q1 Max (ANSI, knob)

Default Keychron keymap plus:

- **Shift+Home → End** (the Home key at the bottom of the column under the knob), via a QMK key override in `keymap.c`. Works on every layer; no UI will show it.
- **VIA enabled**, so [launcher.keychron.com](https://launcher.keychron.com) (Chrome, keyboard plugged in via USB) recognizes the board for ordinary remaps without reflashing. Launcher remaps layer on top of this keymap in EEPROM; the key override keeps working regardless.

Reminder: `Fn+Home` is already End in the default Fn layer, and `Fn+Tab` toggles the backlight.

## Build

The brew ARM toolchains are keg-only (not on PATH), so prefix every qmk command:

```sh
cd ~/code/nmonterroso/qmk_firmware
PATH="/opt/homebrew/opt/arm-none-eabi-gcc@8/bin:/opt/homebrew/opt/arm-none-eabi-binutils/bin:$PATH" \
  qmk compile -kb keychron/q1_max/ansi_encoder -km nmonterroso
```

Output: `keychron_q1_max_ansi_encoder_nmonterroso.bin` at the repo root (also in `.build/`).

## Flash

1. Connection switch on the back to **Cable**, then unplug the keyboard.
2. Hold **Esc** while plugging the USB cable back in. Keyboard looks dead (no lights) = DFU bootloader.
3. Either:
   - **CLI** (builds then waits for the bootloader — you can run it before step 2):

     ```sh
     PATH="/opt/homebrew/opt/arm-none-eabi-gcc@8/bin:/opt/homebrew/opt/arm-none-eabi-binutils/bin:$PATH" \
       qmk flash -kb keychron/q1_max/ansi_encoder -km nmonterroso
     ```

   - **QMK Toolbox** (the UI you half-remember): `brew install --cask qmk-toolbox`, open it, load the `.bin` from the repo root, it detects the DFU device ("STM32 DFU device connected"), hit Flash.

Keyboard reboots on the new firmware immediately. OS switch stays on Windows as usual.

## When the build breaks after a long gap (it will)

Homebrew upgrades rot this toolchain. Fixes applied Aug 2026, in likely-recurring order:

- `Could not find module appdirs` → `$(brew --prefix qmk)/libexec/bin/python -m pip install -r requirements.txt` (from repo root). The error message prints the exact command.
- Pillow/`libtiff` dylib load error → same python: `python -m pip install --ignore-installed Pillow`.
- `clang: error: unknown argument '-meabi=5'` → GCC is silently falling back to Apple's assembler because `/opt/homebrew/opt/arm-none-eabi-binutils` (a symlink) got deleted by a brew cleanup. Restore it: `ln -s ../Cellar/arm-none-eabi-binutils/<version> /opt/homebrew/opt/arm-none-eabi-binutils`. Verify with `arm-none-eabi-gcc -print-prog-name=as` → should print a full path, not just `as`.
- `dfu-suffix`/`dfu-util: command not found` → `brew install dfu-util`.
- Nuclear option if brew allows it: `brew reinstall qmk` (may require `brew trust osx-cross/arm` first).

## Updating from Keychron

The Q1 Max is **not** in upstream QMK — it lives on Keychron's fork, branch `wireless_playground` (this branch is based on it):

```sh
git remote add keychron https://github.com/Keychron/qmk_firmware.git   # once
git fetch keychron wireless_playground
git rebase keychron/wireless_playground
```

Then rebuild and reflash.
