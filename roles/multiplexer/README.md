# marcstraube.common.multiplexer

Install and configure terminal multiplexers (tmux, zellij).

## Description

This role installs terminal multiplexers and deploys global system-wide
configurations. Per-user configuration supports the `_users` list pattern with
managed/initial/disabled modes.

- Install tmux and/or zellij (each individually toggleable)
- Deploy global configurations (`/etc/tmux.conf`, `/etc/zellij/config.kdl`)
- Per-user config deployment with config merge support
- Optional TPM (tmux Plugin Manager) clone for per-user tmux configs
- Skip config deployment per multiplexer (for environments with custom dotfiles)

## Requirements

- ansible-core >= 2.17
- `git` on target hosts (only when TPM is enabled for a user)

## Supported Platforms

| Platform                    | tmux | zellij |
|-----------------------------|------|--------|
| Arch Linux                  | yes  | yes    |
| Debian Trixie               | yes  | no     |
| EL 9 (Rocky, Alma, RHEL)    | yes  | no     |
| EL 10 (Rocky, Alma, RHEL)   | yes  | no     |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable               | Default | Description                    |
|------------------------|---------|--------------------------------|
| `multiplexer_enabled`  | `true`  | Enable the multiplexer role    |

### Multiplexers

| Variable                      | Default | Description                             |
|-------------------------------|---------|-----------------------------------------|
| `multiplexer_tmux_enabled`    | `true`  | Enable tmux terminal multiplexer        |
| `multiplexer_zellij_enabled`  | `false` | Enable zellij (Arch only)               |

### Configuration Management

| Variable                            | Default | Description                              |
|-------------------------------------|---------|------------------------------------------|
| `multiplexer_tmux_config_enabled`   | `true`  | Enable global tmux config deployment     |
| `multiplexer_zellij_config_enabled` | `true`  | Enable global zellij config deployment   |

### tmux Configuration

`multiplexer_tmux_config` dict with sensible defaults:

| Key                | Default            | Description                          |
|--------------------|--------------------|--------------------------------------|
| `prefix`           | `'C-a'`            | Prefix key                           |
| `mouse`            | `true`             | Enable mouse support                 |
| `base_index`       | `1`                | Start window numbering at            |
| `pane_base_index`  | `1`                | Start pane numbering at              |
| `history_limit`    | `10000`            | Scrollback buffer size               |
| `escape_time`      | `10`               | Escape time in ms                    |
| `default_terminal` | `'tmux-256color'`  | Terminal type                        |
| `renumber_windows` | `true`             | Renumber windows on close            |

### zellij Configuration

`multiplexer_zellij_config` dict with sensible defaults:

| Key                  | Default     | Description                |
|----------------------|-------------|----------------------------|
| `theme`              | `'default'` | Color theme                |
| `default_layout`     | `'default'` | Default layout             |
| `mouse_mode`         | `true`      | Enable mouse mode          |
| `simplified_ui`      | `false`     | Simplified UI              |
| `pane_frames`        | `true`      | Show pane frames           |
| `copy_on_select`     | `true`      | Copy on select             |
| `scrollback_editor`  | `''`        | External scrollback editor |

### Per-User Configuration

| Variable                          | Default     | Description                                  |
|-----------------------------------|-------------|----------------------------------------------|
| `multiplexer_users`               | `[]`        | Per-user multiplexer config entries          |
| `multiplexer_user_config_mode`    | `'initial'` | Default mode when user entry has no 'mode'   |
| `multiplexer_user_config_merge`   | `true`      | Default config_merge behavior                |
| `multiplexer_user_tpm_enabled`    | `false`     | Default TPM setting for per-user tmux config |

User entry options:

| Key              | Description                                               |
|------------------|-----------------------------------------------------------|
| `username`       | System username (required)                                |
| `mode`           | `managed` / `initial` / `disabled`                        |
| `config_merge`   | Merge with global config (`true`) or standalone (`false`) |
| `tpm_enabled`    | Clone TPM and add run line to user tmux.conf              |
| `tmux_config`    | Per-user tmux config overrides                            |
| `zellij_config`  | Per-user zellij config overrides                          |

## Tags

| Tag                   | Scope                        |
|-----------------------|------------------------------|
| `multiplexer`         | All tasks                    |
| `multiplexer:install` | Package installation         |
| `multiplexer:tmux`    | tmux configuration           |
| `multiplexer:zellij`  | zellij configuration         |
| `multiplexer:users`   | Per-user config deployment   |

## Example Playbook

```yaml
- name: Include multiplexer role
  ansible.builtin.include_role:
    name: marcstraube.common.multiplexer
  tags:
    - multiplexer
  when: multiplexer_enabled | default(true) | bool
```

## Testing

```bash
cd roles/multiplexer
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

Example inventory configuration:

```yaml
# Install tmux with custom prefix, skip zellij
multiplexer_enabled: true
multiplexer_zellij_enabled: false
multiplexer_tmux_config:
  prefix: 'C-b'
  mouse: true
  base_index: 1

# Per-user config with TPM
multiplexer_users:
  - username: 'johndoe'
    mode: 'managed'
    tpm_enabled: true
    tmux_config:
      prefix: 'C-a'
```

## References

- [tmux manual](https://man.openbsd.org/tmux)
- [tmux GitHub](https://github.com/tmux/tmux)
- [TPM — tmux Plugin Manager](https://github.com/tmux-plugins/tpm)
- [zellij documentation](https://zellij.dev/documentation/)
- [zellij GitHub](https://github.com/zellij-org/zellij)
- [zellij configuration reference](https://zellij.dev/documentation/configuration)

## License

MIT

## Author

Marc Straube
