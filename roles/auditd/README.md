# marcstraube.common.auditd

## Description

This role installs, configures, and manages the Linux Audit daemon (auditd).
It provides comprehensive audit rules for compliance frameworks (PCI-DSS,
HIPAA, NIST 800-53, STIG) and supports custom file watches, syscall monitoring,
and syslog integration.

The role handles the auditd service lifecycle correctly: since auditd refuses
`systemctl restart` (due to `RefuseManualStop=yes` on RHEL/Rocky), handlers
use `auditctl --signal` for stop/reload operations.

## Requirements

- **Ansible**: >= 2.17
- **Kernel audit subsystem**: Required (available in all modern kernels)
- **Full VM or bare metal**: Containers lack kernel audit support

## Supported Platforms

| Platform                   | Notes |
|----------------------------|-------|
| Arch Linux                 |       |
| Debian Trixie              |       |
| EL 9 (Rocky, Alma, RHEL)   |       |
| EL 10 (Rocky, Alma, RHEL)  |       |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable                 | Default | Description                         |
|--------------------------|---------|-------------------------------------|
| `auditd_enabled`         | `true`  | Enable the auditd role              |
| `auditd_service_enabled` | `true`  | Enable and start the auditd service |

### Daemon Configuration

| Variable                         | Default                    | Description                   |
|----------------------------------|----------------------------|-------------------------------|
| `auditd_log_file`                | `/var/log/audit/audit.log` | Log file location             |
| `auditd_log_format`              | `ENRICHED`                 | Log format: `RAW`, `ENRICHED` |
| `auditd_log_group`               | `root`                     | Log file group                |
| `auditd_priority_boost`          | `4`                        | Priority boost (0-4)          |
| `auditd_flush`                   | `incremental_async`        | Flush mode                    |
| `auditd_freq`                    | `50`                       | Records between flushes       |
| `auditd_num_logs`                | `10`                       | Number of rotated log files   |
| `auditd_max_log_file`            | `50`                       | Max log file size (MB)        |
| `auditd_max_log_file_action`     | `rotate`                   | Action at max size            |
| `auditd_space_left`              | `75`                       | Space threshold (MB or %)     |
| `auditd_space_left_action`       | `syslog`                   | Action at space threshold     |
| `auditd_admin_space_left`        | `50`                       | Admin threshold (MB or %)     |
| `auditd_admin_space_left_action` | `suspend`                  | Action at admin threshold     |
| `auditd_disk_full_action`        | `suspend`                  | Action on disk full           |
| `auditd_disk_error_action`       | `suspend`                  | Action on disk error          |

### Network / Remote Logging Receiver

| Variable                     | Default   | Description                            |
|------------------------------|-----------|----------------------------------------|
| `auditd_use_libwrap`         | `true`    | Use tcp_wrappers (if compiled in)      |
| `auditd_tcp_listen_port`     | *(unset)* | TCP listen port (set to enable)        |
| `auditd_tcp_listen_queue`    | `5`       | TCP listen queue depth                 |
| `auditd_tcp_max_per_addr`    | `1`       | Max connections per address            |
| `auditd_tcp_client_ports`    | *(unset)* | Client port range (set to enable)      |
| `auditd_tcp_client_max_idle` | `0`       | Client idle timeout (seconds)          |
| `auditd_report_interval`     | *(unset)* | State report interval (audit >= 4.0.3) |

### Remote Logging (audisp-remote)

| Variable                               | Default     | Description                    |
|----------------------------------------|-------------|--------------------------------|
| `auditd_remote_logging_enabled`        | `false`     | Enable remote log forwarding   |
| `auditd_remote_server`                 | `''`        | Remote server (hostname or IP) |
| `auditd_remote_port`                   | `60`        | Remote port                    |
| `auditd_remote_transport`              | `tcp`       | Transport: `tcp`, `krb5`       |
| `auditd_remote_mode`                   | `immediate` | Mode: `immediate`, `forward`   |
| `auditd_remote_heartbeat_timeout`      | `0`         | Heartbeat timeout (seconds)    |
| `auditd_remote_network_failure_action` | `stop`      | Action on network failure      |

See `defaults/main.yml` for the complete list of remote logging variables.

### Firewall

| Variable                   | Default  | Description                              |
|----------------------------|----------|------------------------------------------|
| `auditd_firewalld_enabled` | `false`  | Enable firewalld rule for remote logging |
| `auditd_firewalld_zone`    | `public` | Firewalld zone for audit remote logging  |

When enabled, opens port 60/tcp (or the configured `auditd_remote_port`) in the
specified firewalld zone. This is intended for servers that **receive** remote
audit logs.

### Email Alerts

| Variable               | Default | Description                          |
|------------------------|---------|--------------------------------------|
| `auditd_email_enabled` | `true`  | Show warning for localhost addresses |
| `auditd_email_address` | `root`  | `action_mail_acct` in auditd.conf    |

### Audit Rules

| Variable                   | Default | Description                                |
|----------------------------|---------|--------------------------------------------|
| `auditd_rules_enabled`     | `true`  | Enable audit rules deployment              |
| `auditd_buffer_size`       | `8192`  | Audit buffer size                          |
| `auditd_backlog_wait_time` | `60000` | Backlog wait time (ms)                     |
| `auditd_rules_immutable`   | `true`  | Lock rules (CIS 4.1.17)                    |
| `auditd_failure_mode`      | `1`     | Failure mode (0=silent, 1=printk, 2=panic) |

### Compliance Presets

| Variable         | Default | Description        |
|------------------|---------|--------------------|
| `auditd_pci_dss` | `true`  | PCI-DSS 10.2 rules |
| `auditd_hipaa`   | `true`  | HIPAA rules        |
| `auditd_nist`    | `true`  | NIST 800-53 rules  |
| `auditd_stig`    | `true`  | STIG rules         |

### Syscall Monitoring

| Variable                 | Default | Description              |
|--------------------------|---------|--------------------------|
| `auditd_time_change`     | `true`  | Time changes             |
| `auditd_identity_change` | `true`  | User/group changes       |
| `auditd_network_change`  | `true`  | Network config changes   |
| `auditd_system_locale`   | `true`  | Locale changes           |
| `auditd_mac_policy`      | `true`  | SELinux/AppArmor changes |
| `auditd_logins`          | `true`  | Login/logout events      |
| `auditd_session`         | `true`  | Session initiation       |
| `auditd_perm_mod`        | `true`  | Permission changes       |
| `auditd_access`          | `true`  | Unsuccessful file access |
| `auditd_delete`          | `true`  | File deletions           |
| `auditd_scope`           | `true`  | Sudoers changes          |
| `auditd_actions`         | `true`  | Sudo usage               |
| `auditd_modules`         | `true`  | Kernel modules           |

### File Watches

| Variable                | Default                            | Description           |
|-------------------------|------------------------------------|-----------------------|
| `auditd_watches`        | `[{path: /var/log/auth.log, ...}]` | Built-in file watches |
| `auditd_custom_watches` | `[]`                               | Custom file watches   |

### Plugins

| Variable                       | Default      | Description          |
|--------------------------------|--------------|----------------------|
| `auditd_plugin_syslog_enabled` | `true`       | Enable syslog plugin |
| `auditd_syslog_facility`       | `LOG_LOCAL6` | Syslog facility      |
| `auditd_syslog_priority`       | `LOG_INFO`   | Syslog priority      |

**Deprecated variables** (removed in v2.0.0):

- `auditd_remote_logging` -- use `auditd_remote_logging_enabled`
- `auditd_plugin_syslog` -- use `auditd_plugin_syslog_enabled`

## Tags

| Tag                | Scope                 |
|--------------------|-----------------------|
| `auditd`           | All role tasks        |
| `auditd:install`   | Package installation  |
| `auditd:configure` | Configuration files   |
| `auditd:rules`     | Audit rules           |
| `auditd:remote`    | Remote logging        |
| `auditd:service`   | Service management    |
| `auditd:firewall`  | Firewalld integration |

## Example Playbook

```yaml
- name: Include auditd role
  ansible.builtin.include_role:
    name: marcstraube.common.auditd
  tags:
    - auditd
  when: auditd_enabled | default(true) | bool
```

### Custom Audit Configuration

```yaml
- name: Include auditd role
  ansible.builtin.include_role:
    name: marcstraube.common.auditd
  vars:
    auditd_max_log_file: 100
    auditd_num_logs: 20
    auditd_email_address: 'admin@example.com'
    auditd_space_left_action: 'email'
    auditd_custom_watches:
      - path: '/etc/ssh/sshd_config'
        permissions: 'wa'
        key: 'sshd_config'
  tags:
    - auditd
  when: auditd_enabled | default(true) | bool
```

## Testing

```bash
cd roles/auditd
molecule test
```

Driver: `vagrant` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

### Version Differences

- **audit 3.x vs 4.x**: `auditd.conf` directives are identical. The
  `report_interval` directive is only available in audit >= 4.0.3.
- **`max_log_file_action: exec`**: Only available in audit >= 4.0.3.
- **`space_left_action: halt`**: Deprecated in latest 4.x releases.

## License

MIT

## Author

Marc Straube
