# marcstraube.common.rkhunter

Install and configure [rkhunter](https://rkhunter.sourceforge.net/) (Rootkit Hunter)
for rootkit detection with systemd timer scheduling.

## Description

Installs and configures rkhunter for rootkit, backdoor, and local exploit
detection. Deploys a managed configuration file, sets up a systemd timer for
scheduled scans, and supports per-OS package manager detection.

## Requirements

- ansible-core >= 2.19
- **Arch Linux**: Package in `extra` repository
- **Debian Trixie**: Package in `main` repository
- **Rocky 9**: Package in EPEL repository (requires `epel-release`); the role
  also installs `file`, which the EPEL package does not pull in and without
  which rkhunter skips the script and hidden file checks

All platforms ship rkhunter 1.4.6 (final upstream release).

## Supported Platforms

| Platform                  | Notes                    |
|---------------------------|--------------------------|
| Arch Linux                |                          |
| Debian Trixie             |                          |
| EL 9 (Rocky, Alma, RHEL)  | Requires EPEL repository |

**Note:** Rocky Linux 10 / EL 10 is not supported -- rkhunter is not available
in EPEL 10.

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable           | Default | Description              |
|--------------------|---------|--------------------------|
| `rkhunter_enabled` | `true`  | Enable the rkhunter role |

### Configuration

| Variable                        | Default         | Description                                 |
|---------------------------------|-----------------|---------------------------------------------|
| `rkhunter_update_on_install`    | `true`          | Update signatures on installation           |
| `rkhunter_mirrors_mode`         | `0`             | Mirror mode: 0=any, 1=local, 2=remote       |
| `rkhunter_rotate_mirrors`       | `true`          | Rotate mirror list for load balancing       |
| `rkhunter_web_cmd`              | `''`            | Download command override (empty=auto)      |
| `rkhunter_hash_cmd`             | `'SHA256'`      | Hash function for file checks               |
| `rkhunter_mail_on_warning`      | `''`            | Email address for warnings (empty=disabled) |
| `rkhunter_mail_cmd`             | `'mail -s ...'` | Mail command template                       |
| `rkhunter_report_warnings_only` | `false`         | Only report warnings in check script        |

Warnings do not fail `rkhunter-check.service` (see [Notes](#notes)). With the
default empty `rkhunter_mail_on_warning` nothing alerts on them: they are only
in `/var/log/rkhunter.log` (`rkhunter_log_dir`), the journal of
`rkhunter-check.service` and syslog (`rkhunter_syslog_facility`, default
`authpriv.warning`). Set an address or monitor one of these; the role prints a
warning on every run while the timer is on and the address is empty.

### SSH Checks

| Variable                       | Default | Description                                 |
|--------------------------------|---------|---------------------------------------------|
| `rkhunter_allow_ssh_root_user` | `'no'`  | Must match sshd_config PermitRootLogin      |
| `rkhunter_allow_ssh_prot_v1`   | `2`     | SSH `Protocol` check (0=no, 1=yes, 2=unset) |

`rkhunter_allow_ssh_prot_v1` is the rkhunter option `ALLOW_SSH_PROT_V1`. With
`2` the check accepts an `sshd_config` without the `Protocol` option, which
OpenSSH 7.6 and later no longer has. `0` warns when the option is missing;
use it when `sshd_config` still sets `Protocol 2`, which warns with `2`.

### Whitelisting

| Variable                    | Default | Description                                       |
|-----------------------------|---------|---------------------------------------------------|
| `rkhunter_allowhiddendir`   | `[]`    | Allowed hidden directories                        |
| `rkhunter_allowhiddenfile`  | `[]`    | Allowed hidden files                              |
| `rkhunter_allowdevfile`     | `[]`    | Allowed files in `/dev` (wildcards allowed)       |
| `rkhunter_scriptwhitelist`  | `[]`    | Whitelisted scripts                               |
| `rkhunter_allowprocdelfile` | `[]`    | Whitelisted processes using deleted files         |
| `rkhunter_allowproclisten`  | `[]`    | Whitelisted processes listening on all interfaces |
| `rkhunter_allowipcpid`      | `[]`    | Whitelisted shared memory PIDs                    |
| `rkhunter_allowipcuser`     | `[]`    | Whitelisted shared memory users                   |

The role writes the scripts and hidden files that the distribution packages
ship in addition to `rkhunter_scriptwhitelist` and `rkhunter_allowhiddenfile`,
so the scan does not warn about them. Scripts go in only when they exist:
rkhunter rejects the configuration and stops on a missing `SCRIPTWHITELIST`
path, the same applies to your own entries. When a package update removes a
whitelisted script, the scheduled check fails with exit code 2 until the role
runs again (`--tags rkhunter:configure`). Adding or removing one of the listed
scripts changes the configuration and runs `rkhunter --propupd`.

| Platform      | Scripts in `/usr/bin` and `/usr/sbin`                   | Hidden files                    |
|---------------|---------------------------------------------------------|---------------------------------|
| Arch Linux    | `egrep`, `fgrep`, `ldd`                                 | `/etc/.updated`, krb5 man pages |
| Debian Trixie | `egrep`, `fgrep`, `ldd`, `which.debianutils`, `adduser` | `/etc/.updated`                 |
| EL 9          | `egrep`, `fgrep`, `ldd`                                 | `/etc/.updated`, krb5 man pages |

The krb5 man pages are `/usr/share/man/man5/.k5identity.5.gz` and
`/usr/share/man/man5/.k5login.5.gz`; `/etc/.updated` is written by
`systemd-update-done`.

The Arch Linux script entries are required: without a package manager check
(`PKGMGR=NONE`) every scan runs the script check. On Debian and EL the entries
are defensive: with `PKGMGR=DPKG`/`RPM` the full scan skips the script check for
files whose hash and size match the package database, and checks them only when
that test fails or runs alone (`--enable scripts`).

On hosts with PostgreSQL, its shared memory segments in `/dev/shm` are
reported as suspicious files in `/dev`. Allow them in the inventory of these
hosts:

```yaml
rkhunter_allowdevfile:
  - '/dev/shm/PostgreSQL.*'
```

### Scanning

| Variable                            | Default                       | Description                          |
|-------------------------------------|-------------------------------|--------------------------------------|
| `rkhunter_enable_tests`             | `'ALL'`                       | Tests to enable (space-separated)    |
| `rkhunter_disable_tests`            | `'suspscan hidden_ports ...'` | Tests to disable                     |
| `rkhunter_suspscan_dirs`            | `['/tmp', '/var/tmp']`        | Directories for suspicious file scan |
| `rkhunter_scan_mode_dev`            | `'THOROUGH'`                  | Device scan mode                     |
| `rkhunter_user_fileprop_files_dirs` | `['/etc/rkhunter.conf']`      | Additional tracked files             |
| `rkhunter_skip_inode_check`         | `false`                       | Skip inode change reporting          |
| `rkhunter_auto_x_detect`            | `true`                        | Auto-detect X display manager        |

### Logging

| Variable                     | Default              | Description                          |
|------------------------------|----------------------|--------------------------------------|
| `rkhunter_use_syslog`        | `true`               | Enable syslog logging                |
| `rkhunter_syslog_facility`   | `'authpriv.warning'` | Syslog facility.priority             |
| `rkhunter_append_log`        | `false`              | Append to log instead of overwriting |
| `rkhunter_copy_log_on_error` | `false`              | Copy log with timestamp on warnings  |
| `rkhunter_log_dir`           | `'/var/log'`         | Log directory                        |

### OS Change Detection

| Variable                     | Default | Description                         |
|------------------------------|---------|-------------------------------------|
| `rkhunter_warn_on_os_change` | `true`  | Warn on OS version change           |
| `rkhunter_updt_on_os_change` | `false` | Auto-update properties on OS change |

### Locking

| Variable                | Default | Description             |
|-------------------------|---------|-------------------------|
| `rkhunter_use_locking`  | `false` | Prevent concurrent runs |
| `rkhunter_lock_timeout` | `300`   | Lock timeout in seconds |

### Scheduled Scans

| Variable                          | Default   | Description                        |
|-----------------------------------|-----------|------------------------------------|
| `rkhunter_timer_enabled`          | `true`    | Enable systemd timer               |
| `rkhunter_timer_oncalendar`       | `'daily'` | Timer schedule (OnCalendar format) |
| `rkhunter_timer_randomized_delay` | `'1h'`    | Randomized delay                   |
| `rkhunter_update_before_scan`     | `true`    | Update signatures before scan      |

## Tags

| Tag                  | Scope                        |
|----------------------|------------------------------|
| `rkhunter`           | All role tasks               |
| `rkhunter:install`   | Package installation + BTRFS |
| `rkhunter:configure` | Configuration                |
| `rkhunter:service`   | Systemd timer management     |

## Example Playbook

```yaml
- name: Include rkhunter role
  ansible.builtin.include_role:
    name: marcstraube.common.rkhunter
  tags: [rkhunter]
  when: rkhunter_enabled | default(true) | bool
```

## Testing

```bash
cd roles/rkhunter
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9

## Notes

- Package manager detection is automatic per OS (`DPKG`/`RPM`/`NONE`)
- The systemd service unit includes security hardening directives, but none
  that hides paths the scan checks (`ProtectKernelModules` would hide
  `/usr/lib/modules` and make the kernel module check warn)
- Warnings count as success of `rkhunter-check.service`: rkhunter reports them
  in its log, the journal, syslog and, with `rkhunter_mail_on_warning`, by
  mail; without an address no failed unit alerts on them. A failed unit means
  the check itself failed: `rkhunter-check.sh` exits with 2 when
  `rkhunter --config-check` fails, and when rkhunter exits with 1 (its code for
  warnings and for errors) without writing the scan summary of this run to its
  log, for example after a database file was replaced by a link, with missing
  file properties or on a lock timeout. Neither case sends a mail. The scan
  runs with `--language en`, so this check works with any `LANGUAGE` setting
  of rkhunter
- The configuration is validated with `rkhunter --config-check` before it is
  written; an invalid entry (e.g. a missing `rkhunter_scriptwhitelist` path)
  fails the task and keeps the previous file
- Module loading is blocked in the unit with `CapabilityBoundingSet=~CAP_SYS_MODULE`
  and `SystemCallFilter=~@module` instead of `ProtectKernelModules`
- The first scan after the installation warns once that no copy of the
  passwd and group files exists yet; rkhunter keeps one from then on
- `rkhunter --propupd` runs automatically as a handler when the config changes
- Debian's built-in cron job (`/etc/default/rkhunter`) is disabled in favor of the systemd timer
- BTRFS: NoCOW attribute is set on `/var/lib/rkhunter/db` when on BTRFS filesystem
- Rocky Linux 10 / EL 10 is not supported (rkhunter not available in EPEL 10)

## References

- [rkhunter](https://rkhunter.sourceforge.net/) — rootkit, backdoor, and local exploit scanner
- [rkhunter Project](https://sourceforge.net/projects/rkhunter/) — SourceForge project page and downloads

## License

MIT

## Author

Marc Straube
