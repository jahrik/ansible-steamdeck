# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Role Overview

Top-level meta role that builds a full developer environment on a Steam Deck by composing sub-roles and managing tools directly. Unlike `ansible-arch-workstation`, it excludes X11 window-manager roles (i3-gaps, polybar, urxvt, conky) and `yay`, since SteamOS runs KDE/gamescope and has a read-only rootfs that blocks AUR builds.

### What this role manages

| Task file | What it does |
|---|---|
| `tasks/konsole.yml` | Konsole profile + Catppuccin Mocha color scheme + default profile setting |
| `tasks/tools.yml` | Static-binary CLI tools to `~/.local/bin` (fzf, ripgrep, bat, eza, delta, zoxide, lazygit, fd, gh, direnv, tealdeer/`tldr`, yq, uv+uvx, Go) + tmux config + Catppuccin Mocha theming for bat/delta/fzf/lazygit/btop |
| `tasks/docker.yml` | Podman socket, docker-cli static binary, dind container, Docker Swarm init, dswarm/mtest/docker shims |
| `tasks/kde.yml` | KDE Plasma: Catppuccin Mocha color scheme, wallpaper, bottom panel, aurorae window decoration, cursor, Papirus-Dark icons |

Most tools fetch their latest release via the GitHub API (`uri` → `tag_name`); `go_version`, `papirus_version`, and the catppuccin `*_version` vars are pinned in `defaults/main.yml`. Each install task has a matching uninstall task gated on `not install`.

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
| `go_version` | `1.26.4` | Go toolchain version (tarball from dl.google.com) |
| `docker_cli_version` | `29.5.3` | docker-cli static binary version |
| `dind_image` | `docker:dind` | Docker-in-Podman image |
| `dind_name` | `dind` | Podman container name |
| `dind_host` | `tcp://127.0.0.1:2375` | Docker host used by dswarm and docker-cli checks |
| `catppuccin_kde_version` | `v0.2.7` | catppuccin/kde release (color schemes + aurorae) |
| `catppuccin_kde_color_scheme` | `CatppuccinMochaMauve` | Active KDE color scheme |
| `catppuccin_accent` | `mauve` | Accent reused for lazygit theme + cursor |
| `catppuccin_wallpaper_url` | dark-forest jpg | Wallpaper download URL |
| `catppuccin_aurorae_theme` | `CatppuccinMocha-Modern` | Aurorae window decoration |
| `catppuccin_cursors_version` | `v2.0.0` | catppuccin/cursors release |
| `papirus_version` | `20250501` | Papirus icon theme release tag |

## KDE Theming Design Notes

`tasks/kde.yml` gates its whole block on `kde_tooling.rc == 0` (a `which plasma-apply-colorscheme` check), so KDE tasks skip cleanly in the Docker `default` scenario where the binaries are absent. KDE verify assertions live only in `molecule/localhost/verify.yml`.

**Global Plasma style / look-and-feel is intentionally NOT applied** — catppuccin/kde ships no official Plasma desktop theme, and applying a global look-and-feel would override the color scheme, panel position, and wallpaper this role sets. The color scheme + window decoration + cursor + Papirus icons give a cohesive Catppuccin look without that risk.

The **Restart plasmashell** handler uses `systemctl --user restart plasma-plasmashell.service` — NOT `plasmashell --replace`. On Plasma 6 Wayland, plasmashell is a supervised systemd user service; a detached `--replace` gets reaped when the ansible run exits, crashing the live desktop.

Icons: Papirus-Dark is installed to `~/.local/share/icons` (no official Catppuccin icon set exists) and set as the icon theme.

## Docker/dind Design Notes

Docker tasks are gated on **both** `ansible_distribution_release == 'holo'` and `/run/user/<uid>` existing (proves a real systemd user session). This prevents the tasks from running in Docker containers used by the `default` molecule scenario.

The dind setup runs `docker:dind` under Podman with `--privileged --network=host`. No `--security-opt seccomp=unconfined`. `files/daemon.json` sets `"iptables": false` and `"bridge": "none"` to avoid NAT chain and bridge-network failures inside the Podman container.

Shims deployed to `~/.local/bin/`:

| Shim | Purpose |
|---|---|
| `docker` | Passes all args to `podman` |
| `dswarm` | Runs static `docker-cli` against the dind swarm (`DOCKER_HOST=tcp://127.0.0.1:2375`) |
| `mtest` | Runs `molecule` with Podman socket and venv PATH; clears stale role cache automatically |

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
