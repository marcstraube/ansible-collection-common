# Ansible Role: marcstraube.common.aide

Configure AIDE (Advanced Intrusion Detection Environment) for file integrity monitoring.

## Description

This role installs and configures AIDE to monitor file system integrity across
multiple Linux distributions. It handles version differences between AIDE 0.16
(Rocky 9) and 0.18+ (Arch Linux, Debian, Rocky 10) automatically through
OS-specific variable files.

## Requirements

- **Ansible**: >= 2.15
- **Collections**:
  - `kewlfft.aur` (Arch Linux only, for AUR package installation)

## Supported Platforms

| Platform             | AIDE Version | Package Source | Notes                          |
| -------------------- | ------------ | -------------- | ------------------------------ |
| Arch Linux           | 0.18.x       | AUR            | Requires `aur_builder` user    |
| Debian 12 (Bookworm) | 0.18.x       | apt            | Uses `aideinit`, `_aide` user  |
| Rocky 9 / RHEL 9     | 0.16         | AppStream      | Legacy `database` directive    |
| Rocky 10 / RHEL 10   | 0.18.x       | AppStream      | Modern `database_in` directive |

### Version Differences

AIDE 0.16 (Rocky 9) and 0.18+ (everything else) differ in configuration directives:

| Feature                 | AIDE 0.16         | AIDE 0.18+                   |
| ----------------------- | ----------------- | ---------------------------- |
| Database read directive | `database`        | `database_in`                |
| Log level control       | `verbose` (0-255) | `log_level` + `report_level` |

The role handles this automatically via `vars/Rocky-9.yml` (0.16 overrides) and
`vars/RedHat.yml` (0.18 defaults for all other RHEL-family distros).

## Role Variables

### Role Control

| Variable               | Default | Description                                     |
| ---------------------- | ------- | ----------------------------------------------- |
| `aide_enabled`         | `true`  | Enable/disable the entire role                  |
| `aide_btrfs_subvolume` | `true`  | Create BTRFS subvolume + NoCOW for database dir |

### Database Configuration

| Variable               | Default                     | Description                      |
| ---------------------- | --------------------------- | -------------------------------- |
| `aide_database`        | `/var/lib/aide/aide.db`     | Database file path               |
| `aide_database_new`    | `/var/lib/aide/aide.db.new` | New database path (for updates)  |
| `aide_database_format` | `gzip`                      | Output format: `plain`, `gzip`   |
| `aide_init_db`         | `true`                      | Initialize database on first run |
| `aide_force_init`      | `false`                     | Force database re-initialization |

### Reporting

| Variable                     | Default                         | Description                         |
| ---------------------------- | ------------------------------- | ----------------------------------- |
| `aide_report_level`          | `changed_attributes`            | Report level for AIDE 0.18+         |
| `aide_verbose`               | `5`                             | Verbose level for AIDE 0.16 (0-255) |
| `aide_report_url`            | `file:///var/log/aide/aide.log` | Primary report URL                  |
| `aide_report_urls`           | `[]`                            | Additional report URLs              |
| `aide_log_dir`               | `/var/log/aide`                 | Log directory path                  |
| `aide_report_retention_days` | `90`                            | Report retention in days            |

### Monitoring Rules

| Variable                   | Default | Description                                       |
| -------------------------- | ------- | ------------------------------------------------- |
| `aide_monitor_binaries`    | `true`  | Monitor `/usr/bin`, `/usr/sbin`, `/usr/local/bin` |
| `aide_monitor_libraries`   | `true`  | Monitor `/usr/lib`, `/usr/lib64`                  |
| `aide_monitor_boot`        | `true`  | Monitor `/boot`                                   |
| `aide_monitor_configs`     | `true`  | Monitor `/etc` and critical config files          |
| `aide_monitor_system_dirs` | `true`  | Monitor `/opt`, `/root`                           |
| `aide_monitor_home_dirs`   | `true`  | Monitor `/home` directories                       |
| `aide_monitor_logs`        | `false` | Monitor log files (growing file check)            |

### Custom Rules

| Variable             | Default | Description                                                 |
| -------------------- | ------- | ----------------------------------------------------------- |
| `aide_custom_rules`  | `[]`    | Custom directories to monitor (list of `path`/`rule` dicts) |
| `aide_exclude_paths` | `[]`    | Additional paths to exclude from monitoring                 |

### Scheduled Checks

| Variable                      | Default | Description                                |
| ----------------------------- | ------- | ------------------------------------------ |
| `aide_timer_enabled`          | `true`  | Enable systemd timer for scheduled checks  |
| `aide_timer_oncalendar`       | `daily` | Timer schedule (systemd OnCalendar format) |
| `aide_timer_randomized_delay` | `1h`    | Randomized delay to avoid load spikes      |

### Email Notifications

| Variable                    | Default                              | Description                         |
| --------------------------- | ------------------------------------ | ----------------------------------- |
| `aide_email_enabled`        | `false`                              | Enable email notifications          |
| `aide_email_to`             | `root@localhost`                     | Email recipient                     |
| `aide_email_subject`        | `[AIDE] File Integrity Check Report` | Email subject                       |
| `aide_email_on_change_only` | `true`                               | Only send email if changes detected |

## Tags

| Tag              | Scope                         |
| ---------------- | ----------------------------- |
| `aide`           | All role tasks                |
| `aide:install`   | Package installation          |
| `aide:configure` | Configuration file deployment |
| `aide:database`  | Database initialization       |
| `aide:schedule`  | Systemd timer setup           |

## Example Playbook

```yaml
- name: Configure AIDE
  hosts: all
  become: true
  roles:
    - role: marcstraube.common.aide
      vars:
        aide_timer_oncalendar: '*-*-* 04:00:00'
        aide_monitor_home_dirs: true
        aide_custom_rules:
          - path: '/opt/myapp'
            rule: 'R+sha512'
        aide_exclude_paths:
          - '/var/lib/docker'
```

## Testing

```bash
cd roles/aide
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9

## License

MIT

## Author

Marc Straube
