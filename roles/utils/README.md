# marcstraube.common.utils

Install common system utilities: process monitors, terminal multiplexers, file transfer
tools, system info fetchers, and fun terminal toys.

## Description

This is an install-only role with no configuration or services. Each utility is gated by a
boolean toggle and mapped to OS-specific package names. Unavailable packages (empty string
in vars) are silently skipped.

## Requirements

- `kewlfft.aur` collection (Arch Linux AUR packages only)
- EPEL repository enabled on Rocky/RHEL (for btop, cowsay, figlet, etc.)

## Supported Platforms

- Arch Linux
- Debian Trixie
- Rocky Linux 9 / 10 (EPEL required)

Package availability varies by platform. See `defaults/main.yml` comments and `vars/` files for details.

## Role Variables

All variables default to `false` unless noted. See `defaults/main.yml` for the complete
list with availability notes per OS.

### Monitors

| Variable               | Default | Description                           |
| ---------------------- | ------- | ------------------------------------- |
| `utils_monitor_htop`   | `true`  | Interactive process viewer            |
| `utils_monitor_btop`   | `false` | Modern resource monitor (btop++)      |
| `utils_monitor_bottom` | `false` | System monitor (Arch only)            |
| `utils_monitor_atop`   | `false` | Advanced process monitor with logging |

### Terminal Multiplexers

| Variable           | Default | Description                        |
| ------------------ | ------- | ---------------------------------- |
| `utils_mux_tmux`   | `true`  | Terminal multiplexer               |
| `utils_mux_zellij` | `false` | Rust-based multiplexer (Arch only) |
| `utils_mux_screen` | `false` | GNU Screen                         |

### File Transfer

| Variable               | Default | Description            |
| ---------------------- | ------- | ---------------------- |
| `utils_transfer_rsync` | `true`  | Fast file copying tool |

### Search Tools

| Variable               | Default | Description                                 |
| ---------------------- | ------- | ------------------------------------------- |
| `utils_search_ripgrep` | `false` | Fast grep replacement (rg)                  |
| `utils_search_fd`      | `false` | Fast find replacement (fd/fdfind on Debian) |

### System Info

| Variable                | Default | Description                                 |
| ----------------------- | ------- | ------------------------------------------- |
| `utils_fetch_fastfetch` | `false` | Fast system info tool (C)                   |
| `utils_fetch_hyfetch`   | `false` | Maintained neofetch successor (Arch/Debian) |
| `utils_fetch_archey4`   | `false` | System info script (Arch AUR only)          |
| `utils_fetch_cpufetch`  | `false` | CPU info tool (Arch only)                   |

### Fun Tools

| Variable                         | Default | Description                            |
| -------------------------------- | ------- | -------------------------------------- |
| `utils_fun_cowsay`               | `false` | Talking cow                            |
| `utils_fun_fortune`              | `false` | Random fortune cookies                 |
| `utils_fun_lolcat`               | `false` | Rainbow text (Arch/Debian)             |
| `utils_fun_ponysay`              | `false` | Pony cowsay (Arch/Debian)              |
| `utils_fun_figlet`               | `false` | Large text banners                     |
| `utils_fun_toilet`               | `false` | Colored text banners                   |
| `utils_fun_sl`                   | `false` | Steam locomotive                       |
| `utils_fun_cmatrix`              | `false` | Matrix screensaver (Arch/Debian)       |
| `utils_fun_cbonsai`              | `false` | Bonsai tree (Arch AUR/Debian)          |
| `utils_fun_asciiquarium`         | `false` | ASCII aquarium (Arch/Debian)           |
| `utils_fun_nyancat`              | `false` | Nyan Cat (Arch/Debian)                 |
| `utils_fun_no_more_secrets`      | `false` | Sneakers effect (Arch AUR/Debian)      |
| `utils_fun_tty_clock`            | `false` | Terminal clock (Arch AUR/Debian)       |
| `utils_fun_pipes`                | `false` | Animated pipes (Arch AUR only)         |
| `utils_fun_pokemon_colorscripts` | `false` | Pokemon scripts (Arch AUR only)        |
| `utils_fun_pokemonsay`           | `false` | Pokemon cowsay (Arch AUR only)         |
| `utils_fun_hollywood`            | `false` | Fake hacker terminal (Arch AUR/Debian) |

## Tags

| Tag             | Scope                        |
| --------------- | ---------------------------- |
| `utils`         | All role tasks               |
| `utils:vars`    | OS-specific variable loading |
| `utils:install` | Package installation         |

## Example Playbook

```yaml
- name: Install system utilities
  hosts: all
  become: true
  tasks:
    - name: Include utils role
      ansible.builtin.include_role:
        name: marcstraube.common.utils
      tags: [utils]
      when: utils_enabled | default(true) | bool
```

## Testing

```bash
cd roles/utils
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## License

MIT

## Author

Marc Straube
