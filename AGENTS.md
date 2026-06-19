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

Each role detects `/etc/steamos-release` and installs to `~/.local` instead of using `pacman`, so the playbook needs no `become`/sudo.

## Key Variables (`defaults/main.yml`)

| Variable | Default | Description |
|---|---|---|
| `install` | `true` | Set to `false` to uninstall |
| `editor` | `nvim` | Default `$EDITOR` (written to zsh export config) |
| `lang` | `en_US.UTF-8` | Locale / `$LANG` (written to zsh export config) |

## Commands

```bash
# Install Galaxy role dependencies
ansible-galaxy install -r requirements.yml

# Run the full playbook (intended to run directly on the Deck)
ansible-playbook playbook.yml -i inventory.ini

# Uninstall
ansible-playbook playbook.yml -i inventory.ini -e install=false

# Lint
yamllint .
ansible-lint

# Test with Molecule (Docker, Arch — exercises the non-SteamOS code path)
molecule test

# Test against this machine directly (only meaningful when run on an actual Steam Deck)
molecule test -s localhost
```

## Molecule Scenarios

- `default` — Docker, Arch Linux (`jahrik/docker-archlinux-ansible`)
- `localhost` — applies the role to the current machine via `ansible_connection: local`
