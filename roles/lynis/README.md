# marcstraube.common.lynis

Install and configure [Lynis](https://cisofy.com/lynis/) for automated security auditing with systemd timer scheduling.

## Description

Lynis is a security auditing tool for Linux systems. This role installs lynis
from OS packages or git (for latest upstream), deploys a custom profile with
configurable directives, and sets up a systemd timer for scheduled audits with
report rotation and hardening index monitoring. Supports BTRFS NoCOW for the
report directory.

## Requirements

- ansible-core >= 2.17
- EPEL repository on EL 9 (Rocky, Alma, RHEL)
- EL 10 requires `lynis_install_method: 'git'` (not available in EPEL 10)

## Supported Platforms

| Platform                  | Notes                                           |
|---------------------------|-------------------------------------------------|
| Arch Linux                |                                                 |
| Debian Trixie             |                                                 |
| EL 9 (Rocky, Alma, RHEL)  | Requires EPEL                                   |
| EL 10 (Rocky, Alma, RHEL) | No EPEL package, uses git install automatically |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable        | Default | Description           |
|-----------------|---------|-----------------------|
| `lynis_enabled` | `true`  | Enable the lynis role |

### Installation

| Variable               | Default                                 | Description                                     |
|------------------------|-----------------------------------------|-------------------------------------------------|
| `lynis_install_method` | `'package'`                             | `'package'` or `'git'` (EL 10 requires `'git'`) |
| `lynis_git_repo`       | `'https://github.com/CISOfy/lynis.git'` | Git repo URL                                    |
| `lynis_install_dir`    | `'/usr/local/lynis'`                    | Git install directory                           |
| `lynis_version`        | `'master'`                              | Git branch/tag                                  |

### Profile Configuration

| Variable                    | Default    | Description                                  |
|-----------------------------|------------|----------------------------------------------|
| `lynis_machine_role`        | `'server'` | `'personal'`, `'workstation'`, or `'server'` |
| `lynis_scan_mode`           | `'normal'` | Scan depth: `'light'`, `'normal'`, `'full'`  |
| `lynis_auditor`             | `''`       | Auditor name in reports                      |
| `lynis_colors`              | `true`     | Colored output                               |
| `lynis_quick_mode`          | `false`    | Quick mode                                   |
| `lynis_skip_plugins`        | `false`    | Disable all plugins                          |
| `lynis_verbose`             | `false`    | Verbose output                               |
| `lynis_debug`               | `false`    | Debug mode                                   |
| `lynis_pause_between_tests` | `0`        | Seconds between tests                        |
| `lynis_skip_tests`          | `[]`       | List of test IDs to skip                     |
| `lynis_plugins`             | `[]`       | List of plugins to enable                    |
| `lynis_show_warnings_only`  | `false`    | Show warnings only                           |

### Compliance

| Variable                     | Default | Description                                      |
|------------------------------|---------|--------------------------------------------------|
| `lynis_compliance_enabled`   | `false` | Enable compliance annotations                    |
| `lynis_compliance_standards` | `[]`    | Standards: `pci-dss`, `hipaa`, `iso27001`, `cis` |

### Reporting

| Variable                     | Default                | Description                              |
|------------------------------|------------------------|------------------------------------------|
| `lynis_log_file`             | `'/var/log/lynis.log'` | Runtime log path                         |
| `lynis_report_dir`           | `'/var/log/lynis'`     | Timestamped report archive directory     |
| `lynis_keep_reports`         | `30`                   | Days to keep old reports                 |
| `lynis_show_hardening_index` | `true`                 | Check hardening index post-audit         |
| `lynis_min_hardening_index`  | `80`                   | Minimum acceptable hardening index       |
| `lynis_email_report`         | `''`                   | Email address for reports (requires MTA) |
| `lynis_upload_results`       | `false`                | Upload to Lynis Enterprise               |

### Custom Tests

| Variable                     | Default               | Description                              |
|------------------------------|-----------------------|------------------------------------------|
| `lynis_custom_tests_enabled` | `false`               | Deploy and enable custom tests directory |
| `lynis_custom_tests_dir`     | `'/etc/lynis/custom'` | Directory for custom test plugins        |

### Scheduled Audits

| Variable                       | Default   | Description                        |
|--------------------------------|-----------|------------------------------------|
| `lynis_timer_enabled`          | `true`    | Enable systemd timer               |
| `lynis_timer_oncalendar`       | `'daily'` | Timer schedule (OnCalendar format) |
| `lynis_timer_randomized_delay` | `'1h'`    | Randomized delay                   |

## Tags

| Tag               | Scope                       |
|-------------------|-----------------------------|
| `lynis`           | All role tasks              |
| `lynis:install`   | Package installation, BTRFS |
| `lynis:configure` | Configuration files         |
| `lynis:service`   | Systemd timer management    |

## Example Playbook

```yaml
- name: Include lynis role
  ansible.builtin.include_role:
    name: marcstraube.common.lynis
  tags: [lynis]
  when: lynis_enabled | default(true) | bool
```

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
- EL 10: lynis is not available in EPEL 10; the role uses git install (`lynis_install_method: 'git'`)

## License

MIT

## Author

Marc Straube
