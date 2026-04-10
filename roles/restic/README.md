# marcstraube.common.restic

Configure automated backups using [restic](https://restic.net/) with systemd timers.

## Description

- Package or binary installation
- Multiple backup jobs with independent schedules
- Automatic repository initialization
- Retention policies with forget/prune
- Pre/post backup hooks
- Healthcheck ping support (healthchecks.io etc.)
- Email notifications on failure
- Optional file-based logging (default: journald)
- systemd service hardening
- BTRFS NoCOW for log directory (when file logging enabled)

## Requirements

- Rocky Linux requires EPEL to be enabled before running this role.
  EPEL is managed by the `marcstraube.common.package_management` role.

## Supported Platforms

| Platform       | Version | Source               |
| -------------- | ------- | -------------------- |
| Arch Linux     | rolling | `restic` (community) |
| Debian Trixie  | 0.18.0  | `restic` (apt)       |
| Rocky Linux 9  | EPEL    | `restic` (EPEL)      |
| Rocky Linux 10 | EPEL    | `restic` (EPEL)      |

## Role Variables

### Service Control

| Variable         | Default | Description         |
| ---------------- | ------- | ------------------- |
| `restic_enabled` | `true`  | Enable/disable role |

### Installation

| Variable                      | Default                   | Description                                   |
| ----------------------------- | ------------------------- | --------------------------------------------- |
| `restic_install_from_package` | `true`                    | Use package manager (false = download binary) |
| `restic_binary_version`       | `'0.18.1'`                | Binary version (when not using packages)      |
| `restic_binary_path`          | `'/usr/local/bin/restic'` | Binary install path                           |

### Backup Jobs

| Variable      | Default | Description                       |
| ------------- | ------- | --------------------------------- |
| `restic_jobs` | `[]`    | List of backup job configurations |

Each job supports:

```yaml
restic_jobs:
  - name: 'system'                    # Job name (used in script/timer names)
    repository: '/backup/restic'      # Repository path/URL
    password: 'vault-reference'       # Repository password
    # password_file: '/path/to/file'  # Or password file
    paths:                            # Paths to back up
      - '/etc'
      - '/home'
    excludes:                         # Exclude patterns
      - '*.tmp'
      - '.cache'
    # exclude_file: '/etc/restic/excludes'
    schedule: '*-*-* 02:00:00'        # systemd OnCalendar format
    retention:                        # Retention policy
      keep_last: 7
      keep_daily: 7
      keep_weekly: 4
      keep_monthly: 6
      keep_yearly: 2
      keep_within: '30d'
    environment:                      # Cloud backend credentials
      AWS_ACCESS_KEY_ID: 'key'
      AWS_SECRET_ACCESS_KEY: 'secret'
    pre_backup: |                     # Pre-backup commands
      systemctl stop myapp
    post_backup: |                    # Post-backup commands
      systemctl start myapp
    backup_options: '--exclude-caches --one-file-system'
```

### Global Settings

| Variable                     | Default             | Description                              |
| ---------------------------- | ------------------- | ---------------------------------------- |
| `restic_user`                | `'root'`            | User to run backups as                   |
| `restic_group`               | `'root'`            | Group for backup files                   |
| `restic_config_dir`          | `'/etc/restic'`     | Configuration directory                  |
| `restic_script_dir`          | `'/usr/local/bin'`  | Backup script directory                  |
| `restic_log_to_file`         | `false`             | Log to file (default: journald)          |
| `restic_log_dir`             | `'/var/log/restic'` | Log directory (when log_to_file enabled) |
| `restic_default_retention`   | see defaults        | Default retention policy                 |
| `restic_default_schedule`    | `'*-*-* 02:00:00'`  | Default timer schedule                   |
| `restic_auto_init`           | `true`              | Auto-initialize repositories             |
| `restic_run_after_configure` | `false`             | Run backup after configuration           |
| `restic_nice_level`          | `10`                | Process priority (0-19)                  |
| `restic_ionice_class`        | `3`                 | IO scheduling class (0-3, 3=idle)        |

### Monitoring

| Variable                     | Default               | Description              |
| ---------------------------- | --------------------- | ------------------------ |
| `restic_healthcheck_enabled` | `false`               | Enable healthcheck pings |
| `restic_healthcheck_url`     | `''`                  | Healthcheck URL          |
| `restic_email_on_failure`    | `false`               | Send email on failure    |
| `restic_email_recipient`     | `''`                  | Email recipient          |
| `restic_email_sender`        | `'restic@{{ fqdn }}'` | Email sender             |

## Tags

| Tag                | Scope                             |
| ------------------ | --------------------------------- |
| `restic`           | All restic tasks                  |
| `restic:install`   | Package/binary installation       |
| `restic:configure` | Configuration, BTRFS, backup jobs |
| `restic:service`   | Timer enable/start, repo init     |

## Example Playbook

```yaml
- name: Backup Configuration
  hosts: all
  become: true
  tags: [backup, never]

  tasks:
    - name: Include restic role
      ansible.builtin.include_role:
        name: marcstraube.common.restic
      tags: [backup, restic]
      when: restic_enabled | default(false) | bool
```

## Logging

By default, backup output goes to the systemd journal. View with:

```bash
journalctl -u restic-backup-<job-name>.service
```

Set `restic_log_to_file: true` to additionally write logs to
`/var/log/restic/<job-name>.log`.

## Manual Operations

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
