# Steam Deck

[![CICD](https://github.com/jahrik/ansible-steamdeck/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-steamdeck/actions/workflows/cicd.yml)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-jahrik.steamdeck-blue?logo=ansible)](https://galaxy.ansible.com/ui/standalone/roles/jahrik/steamdeck/)

Configures a Steam Deck developer environment with a consistent Catppuccin Mocha look across the terminal and desktop. Composes [nerd_fonts](https://github.com/jahrik/ansible-nerd-fonts), [alacritty](https://github.com/jahrik/ansible-alacritty), [zsh](https://github.com/jahrik/ansible-zsh), and [nvim](https://github.com/jahrik/ansible-nvim), then layers on:

- **Terminal & Konsole** — Catppuccin Mocha Konsole profile, font size 18, unlimited scrollback.
- **CLI tools** — fzf, ripgrep, bat, eza, delta, zoxide, lazygit, fd, gh, direnv, tealdeer (`tldr`), yq (jq/btop/tmux come from SteamOS). Catppuccin Mocha theming for bat, delta, fzf, lazygit, and btop.
- **Languages & containers** — uv, Go, and a Podman + dind Docker/Swarm testing stack (`dswarm`/`mtest`/`docker` shims).
- **KDE Plasma** — Catppuccin Mocha (Mauve) color scheme, dark wallpaper, bottom panel, Catppuccin window decorations + cursor, and Papirus-Dark icons. (KDE tasks run only on real hardware.)

Everything works within SteamOS's read-only rootfs: binaries to `~/.local/bin`, configs to `~/.config`, themes to `~/.local/share`. No `pacman`, no sudo.

<!-- vim-markdown-toc GFM -->

* [Requirements](#requirements)
* [Tools](#tools)
* [Role Variables](#role-variables)
  * [General](#general)
  * [Konsole](#konsole)
  * [Docker / dind](#docker--dind)
  * [KDE Plasma](#kde-plasma)
* [Dependencies](#dependencies)
* [Example Playbook](#example-playbook)
* [License](#license)
* [Notes](#notes)
  * [Desktop Mode](#desktop-mode)
  * [Install Ansible](#install-ansible)
  * [Clone and Run](#clone-and-run)
  * [Uninstall](#uninstall)

<!-- vim-markdown-toc -->

## Requirements

Run directly on a Steam Deck in desktop mode (SteamOS). No sudo required — everything installs to `~/.local`.

## Tools

CLI tools installed to `~/.local/bin` (`jq`, `btop`, and `tmux` ship with SteamOS):

| Tool | What it's for |
|---|---|
| `fzf` | Fuzzy finder |
| `rg` (ripgrep) | Fast recursive search |
| `fd` | Fast file find |
| `bat` | `cat` with syntax highlighting |
| `eza` | Modern `ls` (aliased to `ls`/`ll`/`la`) |
| `delta` | Syntax-highlighted git diffs |
| `zoxide` | Smarter `cd` |
| `lazygit` | Git TUI |
| `gh` | GitHub CLI |
| `direnv` | Per-directory environments |
| `tldr` (tealdeer) | Quick command examples |
| `yq` | YAML/JSON processor |
| `uv` | Python package & runtime manager |
| `go` | Go toolchain |

`bat`, `delta`, `fzf`, `lazygit`, and `btop` are themed Catppuccin Mocha.

Each tool is pinned to a specific version in [`defaults/main.yml`](defaults/main.yml) and its download is verified against a SHA256 checksum. To upgrade a tool, bump its `*_version` and `*_checksum` together (the checksum is `sha256sum` of the release asset).

## Role Variables

The handiest knobs are below. See [`defaults/main.yml`](defaults/main.yml) for the complete list — including the full set of `konsole_*` appearance options (line spacing, cursor shape, scrollbar, etc.) and the `catppuccin_*`/`papirus_*` version pins.

### General

| Variable | Default | Description |
|---|---|---|
| `install` | `true` | Set to `false` to uninstall everything |
| `editor` | `nvim` | Sets `$EDITOR` in zsh config |
| `lang` | `en_US.UTF-8` | Sets `$LANG` in zsh config |

### Konsole

| Variable | Default | Description |
|---|---|---|
| `konsole_color_scheme` | `Catppuccin Mocha` | Color scheme (downloaded from catppuccin/konsole) |
| `konsole_font_size` | `18` | Font size in points |
| `konsole_history_mode` | `2` | 1=fixed size, 2=unlimited scrollback |

### Docker / dind

| Variable | Default | Description |
|---|---|---|
| `docker_cli_version` | `29.5.3` | docker-cli static binary version |
| `dind_host` | `tcp://127.0.0.1:2375` | Docker host URL used by dswarm/docker-cli |

### KDE Plasma

| Variable | Default | Description |
|---|---|---|
| `catppuccin_kde_color_scheme` | `CatppuccinMochaMauve` | Active KDE color scheme |
| `catppuccin_accent` | `mauve` | Accent reused for lazygit theme + cursor |
| `catppuccin_wallpaper_url` | dark-forest | Wallpaper download URL |

## Dependencies

| Role | CI |
|------|----|
| jahrik.nerd_fonts | [![CI](https://github.com/jahrik/ansible-nerd-fonts/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-nerd-fonts/actions/workflows/cicd.yml) |
| jahrik.alacritty | [![CI](https://github.com/jahrik/ansible-alacritty/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-alacritty/actions/workflows/cicd.yml) |
| jahrik.zsh | [![CI](https://github.com/jahrik/ansible-zsh/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-zsh/actions/workflows/cicd.yml) |
| jahrik.nvim | [![CI](https://github.com/jahrik/ansible-nvim/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-nvim/actions/workflows/cicd.yml) |

## Example Playbook

```yaml
---
- hosts: localhost
  roles:
    - role: jahrik.steamdeck
```

To uninstall:

```yaml
---
- hosts: localhost
  vars:
    install: false
  roles:
    - role: jahrik.steamdeck
```

## License

GPLv2

## Usage

Switch the Steam Deck to Desktop Mode (hold the power button → **Switch to Desktop**) and open Konsole.

Install Ansible into a user venv (SteamOS's rootfs is read-only):

```bash
python3 -m venv ~/.venv/ansible
source ~/.venv/ansible/bin/activate
pip install ansible
```

Clone the repo and run the playbook:

```bash
git clone https://github.com/jahrik/ansible-steamdeck.git
cd ansible-steamdeck
ansible-galaxy install -r requirements.yml
ansible-playbook playbook.yml -i inventory.ini
```

To uninstall:

```bash
ansible-playbook playbook.yml -i inventory.ini -e install=false
```

## Testing

```bash
uv sync
source .venv/bin/activate
yamllint .
ansible-lint
molecule test
```
