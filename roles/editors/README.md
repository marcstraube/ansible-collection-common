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

- ansible-core >= 2.17
- No additional collections required

## Supported Platforms

| Platform                  | Notes                                  |
|---------------------------|----------------------------------------|
| Arch Linux                |                                        |
| Debian Trixie             |                                        |
| EL 9 (Rocky, Alma, RHEL)  | Neovim via GitHub binary (not in EPEL) |
| EL 10 (Rocky, Alma, RHEL) | Neovim via GitHub binary (not in EPEL) |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable          | Default | Description             |
|-------------------|---------|-------------------------|
| `editors_enabled` | `true`  | Enable the editors role |

### Editors

| Variable                 | Default  | Description                               |
|--------------------------|----------|-------------------------------------------|
| `editors_nano_enabled`   | `true`   | Enable nano editor                        |
| `editors_vim_enabled`    | `true`   | Enable vim editor                         |
| `editors_neovim_enabled` | `true`   | Enable neovim editor                      |
| `editors_default`        | `'nvim'` | Default editor for EDITOR/VISUAL env vars |

### Configuration Management

| Variable                        | Default | Description                            |
|---------------------------------|---------|----------------------------------------|
| `editors_nano_config_enabled`   | `true`  | Enable global nano config deployment   |
| `editors_vim_config_enabled`    | `true`  | Enable global vim config deployment    |
| `editors_neovim_config_enabled` | `true`  | Enable global neovim config deployment |

### Neovim Binary Install (RedHat only)

| Variable                        | Default     | Description                              |
|---------------------------------|-------------|------------------------------------------|
| `editors_neovim_binary_enabled` | `false`     | Enable neovim binary install from GitHub |
| `editors_neovim_version`        | `'v0.10.4'` | Neovim release version to install        |

### Editor Configuration

Each editor has a config dict with sensible defaults. See `defaults/main.yml` for
the full list of options:

- `editors_nano_config` — nano settings (autoindent, linenumbers, tabsize, etc.)
- `editors_vim_config` — vim settings (syntax, numbers, search, visual, etc.)
- `editors_neovim_config` — neovim settings (same as vim plus termguicolors, signcolumn, etc.)

### Per-User Configuration

| Variable                    | Default     | Description                                |
|-----------------------------|-------------|--------------------------------------------|
| `editors_users`             | `[]`        | Per-user editor config entries             |
| `editors_user_config_mode`  | `'managed'` | Default mode when user entry has no 'mode' |
| `editors_user_config_merge` | `true`      | Default config_merge behavior              |

### Deprecated Variables

The following variables are deprecated and will be removed in v2.0.0:

| Old Variable                    | New Variable                    |
|---------------------------------|---------------------------------|
| `editors_nano`                  | `editors_nano_enabled`          |
| `editors_vim`                   | `editors_vim_enabled`           |
| `editors_neovim`                | `editors_neovim_enabled`        |
| `editors_nano_config_manage`    | `editors_nano_config_enabled`   |
| `editors_vim_config_manage`     | `editors_vim_config_enabled`    |
| `editors_neovim_config_manage`  | `editors_neovim_config_enabled` |
| `editors_neovim_binary_install` | `editors_neovim_binary_enabled` |

## Tags

| Tag                   | Scope                           |
|-----------------------|---------------------------------|
| `editors`             | All tasks                       |
| `editors:install`     | Package installation            |
| `editors:environment` | EDITOR/VISUAL environment setup |
| `editors:nano`        | Nano configuration              |
| `editors:vim`         | Vim configuration               |
| `editors:neovim`      | Neovim configuration            |
| `editors:users`       | Per-user config deployment      |

## Example Playbook

```yaml
- name: Include editors role
  ansible.builtin.include_role:
    name: marcstraube.common.editors
  tags:
    - editors
  when: editors_enabled | default(true) | bool
```

## Testing

```bash
cd roles/editors
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

Example inventory configuration:

```yaml
# Install editors but skip global vim config (user has own dotfiles)
editors_enabled: true
editors_vim_config_enabled: false
editors_default: 'nvim'

# Rocky Linux: install neovim from binary release
editors_neovim_binary_enabled: true
editors_neovim_version: 'v0.10.4'
```

## License

MIT

## Author

Marc Straube
