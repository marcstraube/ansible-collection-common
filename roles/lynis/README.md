# marcstraube.common.lynis

Install and configure [Lynis](https://cisofy.com/lynis/) for automated security auditing with systemd timer scheduling.

## Description

Lynis is a security auditing tool for Linux systems. This role:

- Installs lynis from OS packages (or git for latest upstream)
- Deploys a custom profile (`/etc/lynis/custom.prf`) with configurable directives
- Sets up a systemd timer for scheduled audits with report rotation
- Checks hardening index post-audit and warns if below threshold
- Supports BTRFS NoCOW for the report directory

## Requirements

- ansible-core >= 2.17
- EPEL repository on Rocky Linux / RHEL

## Supported Platforms

| Platform       | Lynis Version   | Repository |
| -------------- | --------------- | ---------- |
| Arch Linux     | 3.1.6 (rolling) | extra      |
| Debian Trixie  | 3.1.4           | main       |
| Rocky Linux 9  | 3.0.8           | EPEL       |
| Rocky Linux 10 | 3.1.x           | EPEL       |

Rocky Linux requires EPEL to be enabled before running this role.

## Role Variables

### Installation

| Variable               | Default                                 | Description            |
| ---------------------- | --------------------------------------- | ---------------------- |
| `lynis_enabled`        | `true`                                  | Enable the role        |
| `lynis_install_method` | `'package'`                             | `'package'` or `'git'` |
| `lynis_git_repo`       | `'https://github.com/CISOfy/lynis.git'` | Git repo URL           |
| `lynis_install_dir`    | `'/usr/local/lynis'`                    | Git install directory  |
| `lynis_version`        | `'master'`                              | Git branch/tag         |

### Profile Configuration

| Variable                    | Default    | Description                                  |
| --------------------------- | ---------- | -------------------------------------------- |
| `lynis_machine_role`        | `'server'` | `'personal'`, `'workstation'`, or `'server'` |
| `lynis_auditor`             | `''`       | Auditor name in reports                      |
| `lynis_colors`              | `true`     | Colored output                               |
| `lynis_quick_mode`          | `false`    | Quick mode                                   |
| `lynis_skip_plugins`        | `false`    | Disable all plugins                          |
| `lynis_verbose`             | `false`    | Verbose output                               |
| `lynis_debug`               | `false`    | Debug mode                                   |
| `lynis_pause_between_tests` | `0`        | Seconds between tests                        |
| `lynis_skip_tests`          | `[]`       | List of test IDs to skip                     |
| `lynis_plugins`             | `[]`       | List of plugins to enable                    |
| `lynis_scan_mode`           | `'normal'` | Scan depth: `'light'`, `'normal'`, `'full'`  |
| `lynis_show_warnings_only`  | `false`    | Show warnings only                           |

### Compliance

| Variable                     | Default | Description                                      |
| ---------------------------- | ------- | ------------------------------------------------ |
| `lynis_compliance_enabled`   | `false` | Enable compliance annotations                    |
| `lynis_compliance_standards` | `[]`    | Standards: `pci-dss`, `hipaa`, `iso27001`, `cis` |

### Reporting

| Variable                     | Default                | Description                              |
| ---------------------------- | ---------------------- | ---------------------------------------- |
| `lynis_log_file`             | `'/var/log/lynis.log'` | Runtime log path                         |
| `lynis_report_dir`           | `'/var/log/lynis'`     | Timestamped report archive directory     |
| `lynis_keep_reports`         | `30`                   | Days to keep old reports                 |
| `lynis_show_hardening_index` | `true`                 | Check hardening index post-audit         |
| `lynis_min_hardening_index`  | `80`                   | Minimum acceptable hardening index       |
| `lynis_email_report`         | `''`                   | Email address for reports (requires MTA) |
| `lynis_upload_results`       | `false`                | Upload to Lynis Enterprise               |

### Scheduling

| Variable                       | Default   | Description                        |
| ------------------------------ | --------- | ---------------------------------- |
| `lynis_timer_enabled`          | `true`    | Enable systemd timer               |
| `lynis_timer_oncalendar`       | `'daily'` | Timer schedule (OnCalendar format) |
| `lynis_timer_randomized_delay` | `'1h'`    | Randomized delay                   |

### BTRFS

| Variable            | Default | Description                   |
| ------------------- | ------- | ----------------------------- |
| `lynis_btrfs_nocow` | `true`  | Set NoCOW on report directory |

## Example Playbook

```yaml
- name: Security auditing
  hosts: all
  become: true
  roles:
    - role: marcstraube.common.lynis
      vars:
        lynis_machine_role: 'server'
        lynis_min_hardening_index: 75
        lynis_skip_tests:
          - 'BOOT-5122'
        lynis_compliance_enabled: true
        lynis_compliance_standards:
          - 'iso27001'
```

## Tags

| Tag               | Scope                      |
| ----------------- | -------------------------- |
| `lynis`           | All role tasks             |
| `lynis:install`   | Package installation       |
| `lynis:configure` | Configuration files, BTRFS |
| `lynis:service`   | Systemd timer management   |

## Testing

```bash
cd roles/lynis
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

- The audit script uses `lynis audit system --cronjob` for non-interactive execution
- Reports are timestamped and rotated automatically based on `lynis_keep_reports`
- The systemd service runs with `Nice=19` and `IOSchedulingClass=idle` to minimize system impact
- Lynis is not a daemon — no AppArmor, fail2ban, or firewall integration needed

## License

MIT

## Author

Marc Straube
