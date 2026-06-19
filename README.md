# Steam Deck

[![CICD](https://github.com/jahrik/ansible-steamdeck/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-steamdeck/actions/workflows/cicd.yml)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-jahrik.steamdeck-blue?logo=ansible)](https://galaxy.ansible.com/ui/standalone/roles/jahrik/steamdeck/)

Configures a Steam Deck with [nerd_fonts](https://github.com/jahrik/ansible-nerd-fonts), [alacritty](https://github.com/jahrik/ansible-alacritty), [zsh](https://github.com/jahrik/ansible-zsh), and [nvim](https://github.com/jahrik/ansible-nvim). Each role detects SteamOS and works within its read-only rootfs: fonts go to `~/.local/share/fonts`, binaries to `~/.local/bin`, configs to `~/.config`. No `pacman`, no sudo.

<!-- vim-markdown-toc GFM -->

* [Requirements](#requirements)
* [Role Variables](#role-variables)
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

## Role Variables

    install: true    # Set to false to uninstall
    editor: nvim     # Sets $EDITOR in zsh config
    lang: en_US.UTF-8

## Dependencies

| Role | CI |
|------|----|
| jahrik.nerd_fonts | [![CI](https://github.com/jahrik/ansible-nerd-fonts/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-nerd-fonts/actions/workflows/cicd.yml) |
| jahrik.alacritty | [![CI](https://github.com/jahrik/ansible-alacritty/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-alacritty/actions/workflows/cicd.yml) |
| jahrik.zsh | [![CI](https://github.com/jahrik/ansible-zsh/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-zsh/actions/workflows/cicd.yml) |
| jahrik.nvim | [![CI](https://github.com/jahrik/ansible-nvim/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-nvim/actions/workflows/cicd.yml) |

## Example Playbook

    - hosts: localhost
      roles:
         - { role: jahrik.steamdeck, install: true }

## License

GPLv2

## Notes

### Desktop Mode

Switch the Steam Deck to Desktop Mode: hold the power button → **Switch to Desktop**.

Open Konsole (the KDE terminal). Everything below runs in that terminal.

### Install Ansible

SteamOS's rootfs is read-only, so install Ansible into a user venv:

    python3 -m venv ~/.venv/ansible
    source ~/.venv/ansible/bin/activate
    pip install ansible

Activate it each session before running the playbook:

    source ~/.venv/ansible/bin/activate

### Clone and Run

Clone this repo:

    git clone https://github.com/jahrik/ansible-steamdeck.git
    cd ansible-steamdeck

Install Galaxy dependencies:

    ansible-galaxy install -r requirements.yml

Run the playbook against localhost (no sudo needed):

    ansible-playbook playbook.yml -i inventory.ini

### Uninstall

Set `install: false` to remove everything installed by the role:

    ansible-playbook playbook.yml -i inventory.ini -e install=false
