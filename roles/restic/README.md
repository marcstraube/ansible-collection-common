# marcstraube.common.restic

Configure automated backups using [restic](https://restic.net/) with systemd timers.

## Description

Manages restic backup with systemd timers across Arch Linux, Debian, and RHEL-based systems.
Supports package or binary installation, multiple backup jobs with independent schedules,
automatic repository initialization, retention policies with forget/prune, pre/post backup
hooks, healthcheck pings, email notifications on failure, and optional file-based logging.
Includes systemd service hardening and BTRFS NoCOW for log directory.

## Requirements

- ansible-core >= 2.17
- Rocky Linux requires EPEL to be enabled before running this role.
  EPEL is managed by the `marcstraube.common.package_management` role.

## Supported Platforms

| Platform                   | Notes                      |
|----------------------------|----------------------------|
| Arch Linux                 | `restic` from community    |
| Debian Trixie              | `restic` from apt          |
| EL 9 (Rocky, Alma, RHEL)  | `restic` from EPEL         |
| EL 10 (Rocky, Alma, RHEL) | `restic` from EPEL         |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable         | Default | Description              |
|------------------|---------|--------------------------|
| `restic_enabled` | `true`  | Enable the restic role   |

### Installation

| Variable                      | Default                   | Description                              |
|-------------------------------|---------------------------|------------------------------------------|
| `restic_install_from_package` | `true`                    | Use package manager (false = binary)     |
| `restic_binary_version`       | `'0.18.1'`                | Binary version (when not using packages) |
| `restic_binary_path`          | `'/usr/local/bin/restic'` | Binary install path                      |

### Backup Jobs

| Variable      | Default | Description                       |
|---------------|---------|-----------------------------------|
| `restic_jobs` | `[]`    | List of backup job configurations |

Each job supports:

```yaml
restic_jobs:
  - name: 'system'
    repository: '/backup/restic'
    password: 'vault-reference'
    paths:
      - '/etc'
      - '/home'
    excludes:
      - '*.tmp'
      - '.cache'
    schedule: '*-*-* 02:00:00'
    retention:
      keep_last: 7
      keep_daily: 7
      keep_weekly: 4
      keep_monthly: 6
      keep_yearly: 2
      keep_within: '30d'
    environment:
      AWS_ACCESS_KEY_ID: 'key'
      AWS_SECRET_ACCESS_KEY: 'secret'
    pre_backup: |
      systemctl stop myapp
    post_backup: |
      systemctl start myapp
    backup_options: '--exclude-caches --one-file-system'
```

### Global Settings

| Variable                     | Default             | Description                              |
|------------------------------|---------------------|------------------------------------------|
| `restic_user`                | `'root'`            | User to run backups as                   |
| `restic_group`               | `'root'`            | Group for backup files                   |
| `restic_config_dir`          | `'/etc/restic'`     | Configuration directory                  |
| `restic_script_dir`          | `'/usr/local/bin'`  | Backup script directory                  |
| `restic_log_to_file`         | `false`             | Log to file instead of journald          |
| `restic_log_dir`             | `'/var/log/restic'` | Log directory (when log_to_file enabled) |
| `restic_default_retention`   | see defaults        | Default retention policy                 |
| `restic_default_schedule`    | `'*-*-* 02:00:00'` | Default timer schedule                   |
| `restic_auto_init`           | `true`              | Auto-initialize repositories             |
| `restic_run_after_configure` | `false`             | Run backup after configuration           |
| `restic_nice_level`          | `10`                | Process priority (0-19)                  |
| `restic_ionice_class`        | `3`                 | IO scheduling class (0-3, 3=idle)        |

### Monitoring

| Variable                     | Default               | Description                     |
|------------------------------|-----------------------|---------------------------------|
| `restic_healthcheck_enabled` | `false`               | Enable healthcheck pings        |
| `restic_healthcheck_url`     | `''`                  | Healthcheck URL                 |
| `restic_email_on_failure`    | `false`               | Enable email on failure         |
| `restic_email_recipient`     | `''`                  | Email recipient                 |
| `restic_email_sender`        | `'restic@{{ fqdn }}'` | Email sender address            |

## Tags

| Tag                | Scope                             |
|--------------------|-----------------------------------|
| `restic`           | All restic tasks                  |
| `restic:install`   | Package/binary installation       |
| `restic:configure` | Configuration, BTRFS, backup jobs |
| `restic:service`   | Timer enable/start, repo init     |

## Example Playbook

```yaml
- name: Include restic role
  ansible.builtin.include_role:
    name: marcstraube.common.restic
  tags:
    - restic
  when: restic_enabled | default(true) | bool
```

## Notes

### Logging

By default, backup output goes to the systemd journal. View with:

```bash
journalctl -u restic-backup-<job-name>.service
```

Set `restic_log_to_file: true` to additionally write logs to
`/var/log/restic/<job-name>.log`.

### Manual Operations

The backup script supports multiple commands:

```bash
restic-backup-<job> backup     # Run backup
restic-backup-<job> prune      # Forget + prune old snapshots
restic-backup-<job> check      # Check repository integrity
restic-backup-<job> snapshots  # List snapshots
restic-backup-<job> stats      # Show repository statistics
restic-backup-<job> init       # Initialize repository
```

## Testing

```bash
cd roles/restic
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## License

MIT

## Author

Marc Straube
