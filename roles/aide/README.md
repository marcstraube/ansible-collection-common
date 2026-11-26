# marcstraube.common.aide

## Description

This role installs and configures AIDE (Advanced Intrusion Detection Environment)
for file integrity monitoring across multiple Linux distributions. It handles
version differences between AIDE 0.16 (EL 9) and 0.18+ (Arch Linux, Debian,
EL 10) automatically through OS-specific variable files.

## Requirements

- ansible-core >= 2.19
- **Collections**:
  - `kewlfft.aur` (Arch Linux only, for AUR package installation)
- **Mail** (only with `aide_email_enabled: true`): mail(1) and a working MTA.
  The role installs neither, see [Email Notifications](#email-notifications).

## Supported Platforms

| Platform                  | Notes                        |
|---------------------------|------------------------------|
| Arch Linux                | AUR, requires `aur_builder`  |
| Debian Trixie             | Uses `_aide` user            |
| EL 9 (Rocky, Alma, RHEL)  | AIDE 0.16, legacy directives |
| EL 10 (Rocky, Alma, RHEL) | AIDE 0.18+                   |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable       | Default | Description          |
|----------------|---------|----------------------|
| `aide_enabled` | `true`  | Enable the aide role |

### AIDE Configuration

| Variable               | Default                     | Description                     |
|------------------------|-----------------------------|---------------------------------|
| `aide_database`        | `/var/lib/aide/aide.db`     | Database file path              |
| `aide_database_new`    | `/var/lib/aide/aide.db.new` | New database path (for updates) |
| `aide_database_format` | `gzip`                      | Output format: `plain`, `gzip`  |

### Reporting

| Variable                     | Default                         | Description                         |
|------------------------------|---------------------------------|-------------------------------------|
| `aide_report_level`          | `changed_attributes`            | Report level for AIDE 0.18+         |
| `aide_verbose`               | `5`                             | Verbose level for AIDE 0.16 (0-255) |
| `aide_report_url`            | `file:///var/log/aide/aide.log` | Primary report URL                  |
| `aide_report_urls`           | `[]`                            | Additional report URLs              |
| `aide_log_dir`               | `/var/log/aide`                 | Log directory path                  |
| `aide_report_retention_days` | `90`                            | Report retention in days            |

### Monitoring Rules

| Variable                   | Default | Description                                       |
|----------------------------|---------|---------------------------------------------------|
| `aide_monitor_binaries`    | `true`  | Monitor `/usr/bin`, `/usr/sbin`, `/usr/local/bin` |
| `aide_monitor_libraries`   | `true`  | Monitor `/usr/lib`, `/usr/lib64`                  |
| `aide_monitor_boot`        | `true`  | Monitor `/boot`                                   |
| `aide_monitor_configs`     | `true`  | Monitor `/etc` and critical config files          |
| `aide_monitor_system_dirs` | `true`  | Monitor `/opt`, `/root`                           |
| `aide_monitor_home_dirs`   | `true`  | Monitor `/home` directories                       |
| `aide_monitor_logs`        | `false` | Monitor log files (growing file check)            |

### Custom Rules

| Variable             | Default | Description                                                 |
|----------------------|---------|-------------------------------------------------------------|
| `aide_custom_rules`  | `[]`    | Custom directories to monitor (list of `path`/`rule` dicts) |
| `aide_exclude_paths` | `[]`    | Additional paths to exclude from monitoring                 |

### Scheduled Checks

| Variable                      | Default | Description                                |
|-------------------------------|---------|--------------------------------------------|
| `aide_timer_enabled`          | `true`  | Enable systemd timer for scheduled checks  |
| `aide_timer_oncalendar`       | `daily` | Timer schedule (systemd OnCalendar format) |
| `aide_timer_randomized_delay` | `1h`    | Randomized delay to avoid load spikes      |

### Email Notifications

| Variable                    | Default                              | Description                         |
|-----------------------------|--------------------------------------|-------------------------------------|
| `aide_email_enabled`        | `false`                              | Enable email notifications          |
| `aide_email_to`             | `root@localhost`                     | Email recipient                     |
| `aide_email_subject`        | `[AIDE] File Integrity Check Report` | Subject prefix of change mails      |
| `aide_email_on_change_only` | `true`                               | No mail after a clean check         |

`aide_email_enabled: true` needs mail(1) and a working MTA on the host; the role
installs neither. Typical packages for mail(1): `s-nail` on Arch Linux and EL,
`bsd-mailx` or `mailutils` on Debian (both pull in an MTA there). Without a
`mail` command, or when it fails, `aide-check.service` fails with exit code
`70`.

### Initialization

| Variable                  | Default | Description                      |
|---------------------------|---------|----------------------------------|
| `aide_init_db_enabled`    | `true`  | Initialize database on first run |
| `aide_force_init_enabled` | `false` | Force database re-initialization |

## Tags

| Tag              | Scope                         |
|------------------|-------------------------------|
| `aide`           | All role tasks                |
| `aide:install`   | Package installation + BTRFS  |
| `aide:configure` | Configuration file deployment |
| `aide:database`  | Database initialization       |
| `aide:schedule`  | Systemd timer setup           |

## Example Playbook

```yaml
- name: Include aide role
  ansible.builtin.include_role:
    name: marcstraube.common.aide
  tags:
    - aide
  when: aide_enabled | default(true) | bool
```

### Custom Monitoring

```yaml
- name: Include aide role
  ansible.builtin.include_role:
    name: marcstraube.common.aide
  vars:
    aide_timer_oncalendar: '*-*-* 04:00:00'
    aide_monitor_home_dirs: true
    aide_custom_rules:
      - path: '/opt/myapp'
        rule: 'R+sha512'
    aide_exclude_paths:
      - '/var/lib/docker'
  tags:
    - aide
  when: aide_enabled | default(true) | bool
```

## Testing

```bash
cd roles/aide
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

AIDE 0.16 (EL 9) and 0.18+ (everything else) differ in configuration directives:

| Feature                 | AIDE 0.16         | AIDE 0.18+                   |
|-------------------------|-------------------|------------------------------|
| Database read directive | `database`        | `database_in`                |
| Log level control       | `verbose` (0-255) | `log_level` + `report_level` |

The role handles this automatically via `vars/RedHat-9.yml` (0.16 overrides) and
`vars/RedHat.yml` (0.18 defaults for all other RHEL-family distros).

### Exit Codes and Reports

The timer runs `aide-check.service`, which calls `/usr/local/bin/aide-check.sh`.
The script writes each report to `aide_log_dir` and exits with the exit code of
`aide --check`:

| Exit code | Meaning                                                                           | Unit result |
|-----------|-----------------------------------------------------------------------------------|-------------|
| `0`       | No changes                                                                        | success     |
| `1`–`7`   | Changes, added up: `1` added, `2` removed, `4` changed entries                    | success     |
| above `7` | AIDE failed (it defines `14`–`25`, e.g. `17` config error, `18` missing database) | failed      |
| `70`      | The script itself failed (see below)                                              | failed      |

Exit code `70` means that mail(1) is missing or returned an error, or that
writing the report or rotating old reports failed.

With `aide_email_enabled`, changes send a mail with the subject
`<aide_email_subject>: changes detected on <host>` and AIDE errors one with the
fixed subject `AIDE Error: integrity check failed on <host> (exit code <N>)`.
With `aide_email_on_change_only: false`, a clean check also sends
`<aide_email_subject>: no changes on <host>`. Every mail carries the full
report including the output of AIDE.

Changes are the expected result after package updates, so they do not fail the
unit (`SuccessExitStatus=1 2 3 4 5 6 7`). A failed `aide-check.service` means
that the check or its notification did not work.

With the default `aide_email_enabled: false`, changes therefore show up nowhere
but in the journal and the report files: monitoring that only watches the unit
state does not see them. Watch the journal of `aide-check.service` for the line
`AIDE detected changes! Exit code: <N>` (e.g.
`journalctl -u aide-check.service -g 'AIDE detected changes'`) or the reports
`aide-report-*.log` in `aide_log_dir`, or enable the mail.

AIDE compares against the database until it is replaced, so every check reports
the same changes again. After reviewing them, write a new database and move it
in place (paths for the default `aide_database`, `aide_database_new` and
`aide_database_format: gzip`; without gzip the `.gz` suffix is left out):

```bash
# Config: /etc/aide/aide.conf on Debian, /etc/aide.conf elsewhere
aide --update -c /etc/aide.conf
mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz
```

`aide --update` exits with the same change codes `1`–`7` when it finds changes;
that is not an error. A role run with `-e aide_force_init_enabled=true` also
writes a fresh database, without showing the changes first.

## References

- [AIDE](https://aide.github.io/) — Advanced Intrusion Detection Environment project page
- [AIDE on GitHub](https://github.com/aide/aide) — Source code and issue tracker

## License

MIT

## Author

Marc Straube
