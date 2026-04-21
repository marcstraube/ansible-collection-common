# marcstraube.common.rkhunter

Install and configure [rkhunter](https://rkhunter.sourceforge.net/) (Rootkit Hunter)
for rootkit detection with systemd timer scheduling.

## Description

Installs and configures rkhunter for rootkit, backdoor, and local exploit
detection. Deploys a managed configuration file, sets up a systemd timer for
scheduled scans, and supports per-OS package manager detection.

## Requirements

- ansible-core >= 2.17
- **Arch Linux**: Package in `extra` repository
- **Debian Trixie**: Package in `main` repository
- **Rocky 9**: Package in EPEL repository (requires `epel-release`)

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

### SSH Checks

| Variable                       | Default | Description                                 |
|--------------------------------|---------|---------------------------------------------|
| `rkhunter_allow_ssh_root_user` | `'no'`  | Must match sshd_config PermitRootLogin      |
| `rkhunter_allow_ssh_prot_v1`   | `0`     | Allow SSH protocol v1 (0=no, 1=yes, 2=skip) |

### Whitelisting

| Variable                    | Default | Description                                       |
|-----------------------------|---------|---------------------------------------------------|
| `rkhunter_allowhiddendir`   | `[]`    | Allowed hidden directories                        |
| `rkhunter_allowhiddenfile`  | `[]`    | Allowed hidden files                              |
| `rkhunter_allowdevfile`     | `[]`    | Allowed device files                              |
| `rkhunter_scriptwhitelist`  | `[]`    | Whitelisted scripts/binaries                      |
| `rkhunter_allowprocdelfile` | `[]`    | Whitelisted processes using deleted files         |
| `rkhunter_allowproclisten`  | `[]`    | Whitelisted processes listening on all interfaces |
| `rkhunter_allowipcpid`      | `[]`    | Whitelisted shared memory PIDs                    |
| `rkhunter_allowipcuser`     | `[]`    | Whitelisted shared memory users                   |

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
- The systemd service unit includes security hardening directives
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
