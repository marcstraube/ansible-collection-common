# marcstraube.common.logrotate

Manage logrotate global configuration, custom drop-in configs, and systemd timer.

## Description

This role installs logrotate, deploys a global `/etc/logrotate.conf`, manages custom
drop-in configuration files in `/etc/logrotate.d/`, and ensures the systemd timer is
running.

Service-specific logrotate configs (nginx, postfix, etc.) are **not** managed here.
Each service role is responsible for deploying its own logrotate drop-in config, either
directly or via the `logrotate_configs` list variable.

## Requirements

- ansible-core >= 2.17
- systemd-based system (all supported platforms)

## Supported Platforms

- Arch Linux
- Debian Trixie
- Rocky Linux 9 (logrotate 3.18)
- Rocky Linux 10 (logrotate 3.22)

## Role Variables

### Service Control

| Variable            | Default | Description             |
| ------------------- | ------- | ----------------------- |
| `logrotate_enabled` | `true`  | Enable/disable the role |

### Global Configuration

| Variable                  | Default      | Description                                                     |
| ------------------------- | ------------ | --------------------------------------------------------------- |
| `logrotate_frequency`     | `weekly`     | Rotation interval: daily, weekly, monthly, yearly               |
| `logrotate_rotate`        | `4`          | Number of rotated files to keep                                 |
| `logrotate_create`        | `true`       | Create new empty log files after rotation                       |
| `logrotate_dateext`       | `true`       | Use date suffix instead of numeric                              |
| `logrotate_dateformat`    | `-%Y%m%d`    | Date format for rotated files                                   |
| `logrotate_compress`      | `true`       | Compress rotated files                                          |
| `logrotate_delaycompress` | `true`       | Delay compression one cycle                                     |
| `logrotate_notifempty`    | `true`       | Skip empty log files                                            |
| `logrotate_nomail`        | `true`       | Do not mail rotated logs                                        |
| `logrotate_missingok`     | `true`       | Missing logs are not an error                                   |
| `logrotate_sharedscripts` | `true`       | Run scripts once per rotation                                   |
| `logrotate_su`            | `''`         | Run as user/group (e.g. `root adm`)                             |
| `logrotate_olddir`        | `''`         | Move rotated files to directory                                 |
| `logrotate_tabooext`      | OS-dependent | Additional taboo extensions (Arch: `.pacorig .pacnew .pacsave`) |
| `logrotate_maxage`        | `''`         | Remove rotated files older than N days                          |
| `logrotate_minsize`       | `''`         | Minimum size before rotation                                    |
| `logrotate_maxsize`       | `''`         | Maximum size triggering rotation                                |
| `logrotate_size`          | `''`         | Size-based rotation (replaces frequency)                        |

### Custom Configurations

| Variable            | Default | Description                    |
| ------------------- | ------- | ------------------------------ |
| `logrotate_configs` | `[]`    | List of custom drop-in configs |

Each item in `logrotate_configs`:

```yaml
logrotate_configs:
  - name: 'myapp'
    path: '/var/log/myapp/*.log'
    options:
      frequency: 'daily'
      rotate: 7
      compress: true
      delaycompress: true
      missingok: true
      notifempty: true
      create: '0640 myapp myapp'
      sharedscripts: true
      su: 'root adm'
      olddir: '/var/log/myapp/old'
      copytruncate: false
      dateext: true
      dateformat: '-%Y%m%d'
      maxage: 90
      minsize: '1M'
      maxsize: '100M'
    postrotate: |
      systemctl reload myapp > /dev/null 2>&1 || true
    prerotate: ''
```

### Timer Configuration

| Variable                     | Default | Description                                     |
| ---------------------------- | ------- | ----------------------------------------------- |
| `logrotate_timer_oncalendar` | `''`    | Override timer schedule (e.g. `*-*-* 02:00:00`) |

## Tags

| Tag                   | Scope                    |
| --------------------- | ------------------------ |
| `logrotate`           | All role tasks           |
| `logrotate:install`   | Package installation     |
| `logrotate:configure` | Configuration deployment |
| `logrotate:service`   | Timer management         |

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - role: marcstraube.common.logrotate
      vars:
        logrotate_frequency: 'daily'
        logrotate_rotate: 7
        logrotate_compress: true
```

## Testing

```bash
cd roles/logrotate
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

- Rocky 9 ships logrotate 3.18 — some newer directives (e.g. `ignoreduplicates` from
  3.21) are not available. The role only uses directives supported across all platforms.
- Arch Linux adds `.pacorig`, `.pacnew`, `.pacsave` to tabooext automatically via OS vars.
- journald log management is handled by the `marcstraube.common.base` role, not here.

## License

MIT

## Author

Marc Straube
