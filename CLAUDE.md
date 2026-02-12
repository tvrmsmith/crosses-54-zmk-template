# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a ZMK firmware configuration repository for the "Crosses-54" split keyboard. ZMK is open-source wireless keyboard firmware built on Zephyr RTOS.

## Hardware Configuration

- **Board**: nice_nano (nRF52840 microcontroller)
- **Shield**: crosses (split keyboard with 54 keys)
- **Layout**: 54-key columnar stagger with 6 thumb keys
- **Features**: Bluetooth, ZMK Studio support (right half), OLED display support

## Key Files

- `config/crosses.keymap` - Main keymap configuration with layers and behaviors
- `config/crosses.conf` - ZMK configuration options (display settings, etc.)
- `build.yaml` - Defines build targets for left/right halves and settings reset
- `config/west.yml` - West manifest defining ZMK dependencies
- `keymap-drawer/crosses.yaml` - Human-readable keymap representation for visualization
- `keymap_drawer.config.yaml` - Configuration for keymap visualization styling

## Building Firmware

Firmware builds automatically via GitHub Actions on push/PR. The workflow is defined in `.github/workflows/build.yml` and uses ZMK's official build workflow.

Build outputs:
- `crosses_54_left.uf2` - Left half firmware
- `crosses_54_right.uf2` - Right half firmware (with ZMK Studio enabled)
- `settings_reset.uf2` - Reset settings if needed

To flash: Download artifacts from GitHub Actions and drag .uf2 files to the bootloader drive.

## Keymap Visualization

To update the keymap visualization after modifying `config/crosses.keymap`:

```bash
# Step 1: Parse the ZMK keymap to YAML format
keymap parse -z config/crosses.keymap > keymap-drawer/crosses.yaml

# Step 2: Draw the SVG using the physical layout from info.json and the config styling
keymap draw -j config/info.json -l gggw_crosses_54_layout keymap_drawer.config.yaml keymap-drawer/crosses.yaml -o keymap-drawer/crosses.svg
```

**Important details:**
- The keymap CLI tool is available at `~/.local/bin/keymap`
- The `parse` command converts ZMK keymap syntax to keymap-drawer's YAML format
- The `draw` command requires:
  - `-j config/info.json` - QMK-format physical layout definition (key positions)
  - `-l gggw_crosses_54_layout` - Name of the layout within info.json to use
  - `keymap_drawer.config.yaml` - Styling configuration (passed as a YAML file to merge)
  - `keymap-drawer/crosses.yaml` - The parsed keymap data
  - `-o keymap-drawer/crosses.svg` - Output SVG file
- Multiple YAML files can be passed to `draw` and are merged together, allowing the config to override defaults

## Keymap Architecture

The keymap uses a 4-layer design:

1. **Base Layer** - QWERTY with home row mods (Shift/Ctrl/Alt/Gui on home row)
2. **Lower Layer** - Numbers and navigation (arrow keys, Bluetooth controls)
3. **Raise Layer** - Symbols and special characters
4. **Mouse Layer** - Mouse click emulation and ZMK Studio unlock

### Custom Behaviors

- `esc_hyper` - Tap for ESC, hold for Hyper (Ctrl+Shift+Alt+Gui)
- `hml` / `hmr` - Timeless home row mods with "balanced" flavor
  - Left hand mods: positions trigger on right hand typing
  - Right hand mods: positions trigger on left hand typing
  - `require-prior-idle-ms: 150` prevents accidental mod activation during fast typing
  - `hold-trigger-on-release` enables mod behavior after key release

### Macros

- `hyper` - Macro that presses all modifiers simultaneously (Ctrl+Shift+Alt+Gui)

## ZMK Studio

The right half includes ZMK Studio support for live keymap editing via USB. The `&studio_unlock` binding on the Mouse layer (position V on base layer) enables Studio mode.

## Git Configuration Note

This is a personal repository under `tvrmsmith/*`, so git remote should use `github-personal` SSH host alias instead of `github.com`. See global CLAUDE.md for SSH configuration details.
