# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A complete Hyprland desktop environment configuration ("dotfiles") distribution called **illogical-impulse**, centered on:
- **Hyprland** — Wayland compositor
- **Quickshell** (QML/QtQuick) — widget system for bars, sidebars, overlays
- **Material Design 3** — auto-generated color schemes from wallpaper

The repo ships dotfiles under `dots/` and an interactive installer under `setup`.

## Setup commands

```bash
# Full install
./setup install

# Individual steps
./setup install-deps     # Step 1: install system packages
./setup install-setups   # Step 2: configure permissions/services
./setup install-files    # Step 3: copy dots/ → ~/.config

# Other subcommands
./setup exp-update       # Update without full reinstall
./setup exp-merge        # Rebase-based merge of upstream changes
./setup uninstall
./setup resetfirstrun
./setup diagnose         # Gather system info for bug reports
```

Key install flags: `--skip-alldeps`, `--skip-allsetups`, `--skip-allfiles`, `--skip-quickshell`, `--skip-hyprland`, `--core` (minimal, no KDE/fish/fonts), `--fontset <name>`.

There are no lint or test commands — this is configuration, not a compiled project.

## Architecture

### Directory layout

```
dots/               → config files copied verbatim to ~/.config (and ~/.local/share)
dots-extra/         → optional extras (emacs, fcitx5, swaylock, NixOS, font sets)
sdata/              → installer infrastructure
  lib/              → shared bash libraries (env vars, functions, pkg installers, OS detection)
  subcmd-install/   → numbered install steps (0.greeting → 3.files)
  dist-arch/        → Arch-only: PKGBUILDs + dep installer scripts
  dist-fedora/      → community Fedora support
  dist-nix/         → experimental NixOS support
setup               → entry point script; routes to sdata/subcmd-* directories
diagnose            → collects system info for debugging
```

### Hyprland config hierarchy

`~/.config/hypr/hyprland.conf` sources files in two layers:

1. **Defaults** — `hyprland/*.conf` (env, execs, general, rules, colors, keybinds)
2. **User overrides** — `custom/*.conf` (same names)

User changes belong in `custom/` files, never in `hyprland/` defaults.

### Quickshell widget system

Entry point: `dots/.config/quickshell/ii/shell.qml`

```
quickshell/ii/
  shell.qml           → ShellRoot; initializes theme, wallpaper, first-run, etc.
  GlobalStates.qml    → shared state singleton
  settings.qml        → standalone settings window
  modules/
    ii/               → main illogical-impulse widgets (bar, overview, sidebar, etc.)
    common/           → reused components and utilities
    settings/         → settings UI module
    waffle/           → alternative panel family
  services/           → background QML services (DateTime, ResourceUsage, AppSearch, Idle, …)
  panelFamilies/      → top/bottom/side panel definitions
  scripts/            → helper shell scripts
    ai/               → Gemini API + Ollama integration
    colors/           → wallpaper → Material color pipeline (switchwall.sh)
    musicRecognition/
    cava/             → audio visualizer bridge
```

The `panelFamily` property in `shell.qml` switches between `"ii"` and `"waffle"` layouts.

### Installer library conventions (`sdata/lib/`)

- `functions.sh` — interactive wrappers: `v()` shows a command, `x()` runs it with y/e/s prompts, `pause()` waits for user input
- `package-installers.sh` — distro-agnostic `pkg_install`, `pkg_remove`, etc.
- `dist-determine.sh` — sets `$DISTRO` and `$PKG_MANAGER`
- `environment-variables.sh` — XDG paths and ANSI color codes (`$STY_*`)

Install steps in `sdata/subcmd-install/` are numbered scripts (0–3) sourced in sequence by the `install` subcommand.

### Arch meta-packages (`sdata/dist-arch/`)

Dependencies are split into named meta-packages (`illogical-impulse-audio`, `-backlight`, `-basic`, `-fonts-themes`, `-hyprland`, `-kde`, `-portal`, `-python`, `-quickshell-git`, `-widgets`, etc.). Each is a local PKGBUILD. The pinned Quickshell commit lives in `illogical-impulse-quickshell-git/PKGBUILD`.

## Key config files to know

| File | Purpose |
|------|---------|
| `dots/.config/hypr/hyprland.conf` | Sources all other Hyprland configs |
| `dots/.config/hypr/custom/keybinds.conf` | User keybind overrides |
| `dots/.config/hypr/custom/execs.conf` | Autostart programs |
| `dots/.config/hypr/monitors.conf` | Monitor layout (nwg-displays compatible) |
| `dots/.config/quickshell/ii/shell.qml` | Quickshell entry point |
| `dots/.config/quickshell/ii/GlobalStates.qml` | Shared runtime state |
| `sdata/lib/functions.sh` | Core installer utilities |

## Submodule

`dots/.config/quickshell/ii/modules/common/widgets/shapes` is a git submodule (`rounded-polygon-qmljs`). After cloning, run `git submodule update --init` if that directory is empty.
