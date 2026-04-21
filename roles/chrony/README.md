# marcstraube.common.chrony

Configure chrony NTP time synchronization.

## Description

Installs and configures chrony as the NTP daemon, replacing systemd-timesyncd.
Supports client mode (default) and server mode for local networks.

## Requirements

- ansible-core >= 2.17
- `ansible.posix` collection (for firewalld integration)

## Supported Platforms

| Platform                  | Notes                     |
|---------------------------|---------------------------|
| Arch Linux                |                           |
| Debian Trixie             | Includes AppArmor profile |
| EL 9 (Rocky, Alma, RHEL)  |                           |
| EL 10 (Rocky, Alma, RHEL) |                           |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable                 | Default | Description                         |
|--------------------------|---------|-------------------------------------|
| `chrony_enabled`         | `true`  | Enable the chrony role              |
| `chrony_service_enabled` | `true`  | Enable and start the chrony service |

### NTP Sources

| Variable             | Default            | Description                            |
|----------------------|--------------------|----------------------------------------|
| `chrony_ntp_servers` | `0-3.pool.ntp.org` | List of `{server, options}` dicts      |
| `chrony_ntp_pools`   | `[]`               | List of `{pool, options}` dicts        |
| `chrony_minsources`  | `0`                | Minimum sources for sync (0 = default) |

### NTP Server Mode

| Variable                | Default | Description                               |
|-------------------------|---------|-------------------------------------------|
| `chrony_allow_networks` | `[]`    | Networks allowed to query this server     |
| `chrony_deny_networks`  | `[]`    | Networks denied (overrides allow)         |
| `chrony_local_stratum`  | `0`     | Local stratum when offline (0 = disabled) |

### Timing

| Variable                    | Default | Description                                  |
|-----------------------------|---------|----------------------------------------------|
| `chrony_makestep_threshold` | `1.0`   | Step threshold in seconds                    |
| `chrony_makestep_limit`     | `3`     | Number of initial updates to allow stepping  |
| `chrony_max_slew_rate`      | `0`     | Max slew rate in ppm (0 = default)           |
| `chrony_max_offset`         | `0`     | Max offset for maxchange (0 = disabled)      |
| `chrony_maxchange_start`    | `1`     | Updates before maxchange enforcement         |
| `chrony_maxchange_ignore`   | `true`  | Log only (true) or exit (false) on violation |

### RTC

| Variable           | Default | Description                              |
|--------------------|---------|------------------------------------------|
| `chrony_rtcsync`   | `true`  | Sync system time to RTC                  |
| `chrony_rtc_local` | `false` | RTC is in local time (dual-boot Windows) |

### Logging

| Variable             | Default                                | Description        |
|----------------------|----------------------------------------|--------------------|
| `chrony_log_dir`     | `/var/log/chrony`                      | Log directory path |
| `chrony_log_options` | `[measurements, statistics, tracking]` | What to log        |

### Authentication

| Variable               | Default            | Description                  |
|------------------------|--------------------|------------------------------|
| `chrony_keys_file`     | `/etc/chrony.keys` | Key file path                |
| `chrony_generate_keys` | `false`            | Generate key file if missing |

### NTS (Network Time Security)

Client-side NTS: add `nts` to server options (e.g., `options: 'iburst nts'`).

| Variable                    | Default           | Description               |
|-----------------------------|-------------------|---------------------------|
| `chrony_nts_server_enabled` | `false`           | Enable NTS server mode    |
| `chrony_nts_server_cert`    | `''`              | TLS certificate path      |
| `chrony_nts_server_key`     | `''`              | TLS private key path      |
| `chrony_ntsdumpdir`         | `/var/lib/chrony` | NTS cookie dump directory |

### Hardware Timestamping

| Variable                         | Default | Description                    |
|----------------------------------|---------|--------------------------------|
| `chrony_hw_timestamp_interfaces` | `[]`    | Interfaces for HW timestamping |

### Leap Seconds

| Variable             | Default | Description                                    |
|----------------------|---------|------------------------------------------------|
| `chrony_leapsectz`   | `''`    | Timezone with leap seconds (e.g., `right/UTC`) |
| `chrony_leapseclist` | `''`    | Path to leap-seconds.list file                 |

### Security

| Variable                  | Default     | Description              |
|---------------------------|-------------|--------------------------|
| `chrony_bind_cmd_address` | `127.0.0.1` | Bind address for chronyc |

### Firewall

| Variable                   | Default  | Description                       |
|----------------------------|----------|-----------------------------------|
| `chrony_firewalld_enabled` | `false`  | Enable firewalld NTP service rule |
| `chrony_firewalld_zone`    | `public` | firewalld zone                    |

### AppArmor

| Variable                  | Default | Description                            |
|---------------------------|---------|----------------------------------------|
| `chrony_apparmor_enabled` | `false` | Enforce AppArmor profile (Debian/Arch) |

### Extra Configuration

| Variable              | Default | Description                       |
|-----------------------|---------|-----------------------------------|
| `chrony_extra_config` | `[]`    | Additional chrony.conf directives |

## Tags

| Tag                | Scope                |
|--------------------|----------------------|
| `chrony`           | All role tasks       |
| `chrony:install`   | Package installation |
| `chrony:configure` | Configuration        |
| `chrony:service`   | Service management   |
| `chrony:firewall`  | Firewall integration |
| `chrony:apparmor`  | AppArmor enforcement |

## Example Playbook

```yaml
- name: Include chrony role
  ansible.builtin.include_role:
    name: marcstraube.common.chrony
  tags:
    - chrony
  when: chrony_enabled | default(true) | bool
```

### NTP Server for Local Network

```yaml
- name: Include chrony role
  ansible.builtin.include_role:
    name: marcstraube.common.chrony
  vars:
    chrony_allow_networks:
      - '192.168.1.0/24'
    chrony_local_stratum: 10
    chrony_firewalld_enabled: true
  tags:
    - chrony
  when: chrony_enabled | default(true) | bool
```

## Testing

```bash
cd roles/chrony
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## References

- [chrony](https://chrony-project.org/) — Versatile NTP implementation
- [chrony.conf(5)](https://chrony-project.org/doc/4.6/chrony.conf.html) — chrony configuration file reference

## License

MIT

## Author

Marc Straube
