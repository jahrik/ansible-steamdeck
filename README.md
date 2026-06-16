# Steam Deck

[![CICD](https://github.com/jahrik/ansible-steamdeck/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-steamdeck/actions/workflows/cicd.yml)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-jahrik.steamdeck-blue?logo=ansible)](https://galaxy.ansible.com/ui/standalone/roles/jahrik/steamdeck/)

Configures a Steam Deck with [alacritty](https://github.com/jahrik/ansible-alacritty), [zsh](https://github.com/jahrik/ansible-zsh), and [nvim](https://github.com/jahrik/ansible-nvim). All three roles detect SteamOS's read-only rootfs and install to `~/.local` instead of using `pacman`/AUR, so no sudo password is needed.

## Requirements

Run directly on a Steam Deck in desktop mode (SteamOS). No sudo required.

## Example Playbook

```yaml
- hosts: all
  roles:
    - jahrik.steamdeck
```

## Usage

```bash
ansible-galaxy install -r requirements.yml
ansible-playbook playbook.yml -i inventory.ini
```

## License

GPLv2
