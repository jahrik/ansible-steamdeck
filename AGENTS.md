# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Role Overview

Top-level meta role that configures a Steam Deck by composing sub-roles from Ansible Galaxy. Unlike `ansible-arch-workstation`, it deliberately excludes X11 window-manager roles (i3-gaps, polybar, urxvt, conky) and `yay`, since SteamOS runs KDE/gamescope and has a read-only rootfs that blocks AUR builds.

## Composed Roles (`requirements.yml`)

| Role | Purpose |
|---|---|
| `jahrik.nerd_fonts` | Nerd Fonts (DejaVuSansMono, JetBrainsMono) |
| `jahrik.alacritty` | Terminal emulator + config |
| `jahrik.zsh` | Shell + Oh My Zsh + Powerlevel10k |
| `jahrik.nvim` | Neovim + Lazy + LSP |

Each role detects `ansible_distribution_release == 'holo'` (from `VERSION_CODENAME` in `/etc/os-release`) and installs to `~/.local` instead of using `pacman`, so the playbook needs no `become`/sudo. Do not use `ansible_distribution == 'SteamOS'` — on the real Deck, Ansible reads `/etc/arch-release` first and always reports `Archlinux` for that fact.

## Key Variables (`defaults/main.yml`)

| Variable | Default | Description |
|---|---|---|
| `install` | `true` | Set to `false` to uninstall |
| `editor` | `nvim` | Default `$EDITOR` (written to zsh export config) |
| `lang` | `en_US.UTF-8` | Locale / `$LANG` (written to zsh export config) |
| `konsole_profile_name` | `Default` | Konsole profile filename (without `.profile`) |
| `konsole_shell` | `~/.local/bin/zsh` | Shell command set in the Konsole profile |
| `konsole_color_scheme` | `Catppuccin Mocha` | Color scheme name — `catppuccin-mocha.colorscheme` is downloaded from catppuccin/konsole |
| `konsole_font` | `DejaVu Sans Mono` | Font family |
| `konsole_font_size` | `18` | Font size in points |
| `konsole_line_spacing` | `0` | Extra pixels between lines (`[Appearance]`) |
| `konsole_terminal_margin` | `1` | Padding in pixels between border and content |
| `konsole_cursor_shape` | `0` | Cursor shape: 0=block, 1=I-beam, 2=underline |
| `konsole_blinking_cursor` | `false` | Enable cursor blinking |
| `konsole_history_mode` | `2` | Scrollback mode: 1=fixed size, 2=unlimited |
| `konsole_history_size` | `0` | Scrollback line count (unused when `history_mode=2`) |
| `konsole_scroll_bar_position` | `2` | Scrollbar: 0=left, 1=right, 2=hidden |
| `konsole_highlight_scrolled_lines` | `true` | Briefly highlight lines scrolled into view |
| `konsole_auto_copy_selected_text` | `false` | Copy selection to clipboard automatically (X11 style) |
| `konsole_underline_links` | `true` | Underline URLs on hover |

## Commands

```bash
# Install Galaxy role dependencies
ansible-galaxy install -r requirements.yml

# Run the full playbook (intended to run directly on the Deck)
ansible-playbook playbook.yml -i inventory.ini

# Uninstall
ansible-playbook playbook.yml -i inventory.ini -e install=false

# Install test dependencies and activate venv
uv sync
source .venv/bin/activate

# Lint
yamllint .
ansible-lint

# Test with Molecule (Docker, Arch — exercises the non-SteamOS code path)
molecule test

# Test against this machine directly (only meaningful when run on an actual Steam Deck)
molecule test -s localhost
```

## Molecule Scenarios

- `default` — Docker, Arch Linux (`jahrik/docker-archlinux-ansible`) with SteamOS simulated
- `localhost` — applies the role to the current machine via `ansible_connection: local`

## CI

- **Lint**: yamllint + ansible-lint
- **Molecule**: Arch Linux Docker container (SteamOS scenario)
- **Release**: publishes to Ansible Galaxy on merge to `main`
