# marcstraube.common.editors

Install and configure text editors (nano, vim, neovim).

## Description

This role installs text editors and deploys global system-wide configurations.
Per-user editor configuration (dotfiles, plugins, plugin managers) is intentionally
out of scope — users should manage their own editor configs via dotfiles.

- Install nano, vim, and neovim (each individually toggleable)
- Deploy global editor configurations (`/etc/nanorc`, `/etc/vimrc`, `/etc/xdg/nvim/sysinit.vim`)
- Set system-wide `EDITOR`/`VISUAL` via `/etc/profile.d/editor.sh`
- Skip config deployment per editor (for environments with custom dotfiles)
- Neovim binary install from GitHub releases on RedHat (not in EPEL)

## Requirements

- **Ansible**: >= 2.17

## Supported Platforms

| Platform           | Notes                                  |
| ------------------ | -------------------------------------- |
| Arch Linux         | Rolling release                        |
| Debian Trixie (13) |                                        |
| Rocky Linux 9      | Neovim via GitHub binary (not in EPEL) |
| Rocky Linux 10     | Neovim via GitHub binary (not in EPEL) |

## Role Variables

### Control

| Variable          | Default  | Description                               |
| ----------------- | -------- | ----------------------------------------- |
| `editors_enabled` | `true`   | Master toggle for the role                |
| `editors_nano`    | `true`   | Install nano                              |
| `editors_vim`     | `true`   | Install vim                               |
| `editors_neovim`  | `true`   | Install neovim                            |
| `editors_default` | `'nvim'` | Default editor for EDITOR/VISUAL env vars |

### Configuration Management

| Variable                       | Default | Description                 |
| ------------------------------ | ------- | --------------------------- |
| `editors_nano_config_manage`   | `true`  | Deploy global nano config   |
| `editors_vim_config_manage`    | `true`  | Deploy global vim config    |
| `editors_neovim_config_manage` | `true`  | Deploy global neovim config |

Set to `false` to install the editor without touching its global config file.

### Neovim Binary Install (RedHat only)

| Variable                        | Default     | Description                         |
| ------------------------------- | ----------- | ----------------------------------- |
| `editors_neovim_binary_install` | `false`     | Install neovim from GitHub releases |
| `editors_neovim_version`        | `'v0.10.4'` | Neovim release version to install   |

### Editor Configuration

Each editor has a config dict with sensible defaults. See `defaults/main.yml` for
the full list of options:

- `editors_nano_config` — nano settings (autoindent, linenumbers, tabsize, etc.)
- `editors_vim_config` — vim settings (syntax, numbers, search, visual, etc.)
- `editors_neovim_config` — neovim settings (same as vim plus termguicolors, signcolumn, etc.)

## Tags

| Tag                   | Scope                           |
| --------------------- | ------------------------------- |
| `editors`             | All tasks                       |
| `editors:vars`        | OS variable loading             |
| `editors:install`     | Package installation            |
| `editors:environment` | EDITOR/VISUAL environment setup |
| `editors:nano`        | Nano configuration              |
| `editors:vim`         | Vim configuration               |
| `editors:neovim`      | Neovim configuration            |

## Example Playbook

```yaml
- name: Include editors role
  ansible.builtin.include_role:
    name: marcstraube.common.editors
  tags: [base, editors]
  when: editors_enabled | default(true) | bool
```

## Example Inventory

```yaml
# Install editors but skip global vim config (user has own dotfiles)
editors_enabled: true
editors_vim_config_manage: false
editors_default: 'nvim'

# Rocky Linux: install neovim from binary release
editors_neovim_binary_install: true
editors_neovim_version: 'v0.10.4'
```

## Testing

```bash
cd roles/editors
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## License

MIT

## Author

Marc Straube
