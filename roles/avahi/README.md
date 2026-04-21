# marcstraube.common.avahi

Configure Avahi mDNS/DNS-SD service discovery daemon.

## Description

Installs and configures the Avahi daemon for mDNS (multicast DNS) and DNS-SD
(DNS Service Discovery) on the local network. Optionally installs nss-mdns for
`.local` hostname resolution and manages the nsswitch.conf hosts entry.

## Requirements

- ansible-core >= 2.17
- `ansible.posix` collection (for firewalld integration)

## Supported Platforms

| Platform                  | Notes |
|---------------------------|-------|
| Arch Linux                |       |
| Debian Trixie             |       |
| EL 9 (Rocky, Alma, RHEL)  |       |
| EL 10 (Rocky, Alma, RHEL) |       |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable                | Default | Description                        |
|-------------------------|---------|------------------------------------|
| `avahi_enabled`         | `true`  | Enable the avahi role              |
| `avahi_service_enabled` | `true`  | Enable and start the avahi service |

### Server Settings

| Variable                      | Default   | Description                                   |
|-------------------------------|-----------|-----------------------------------------------|
| `avahi_hostname`              | `''`      | Hostname to publish (empty = system hostname) |
| `avahi_domain_name`           | `'local'` | mDNS domain name                              |
| `avahi_browse_domains`        | `[]`      | Additional browse domains                     |
| `avahi_allow_point_to_point`  | `false`   | Allow mDNS on point-to-point interfaces       |
| `avahi_disallow_other_stacks` | `false`   | Exclusively bind UDP 5353                     |

### Network Settings

| Variable                        | Default   | Description                          |
|---------------------------------|-----------|--------------------------------------|
| `avahi_use_ipv4`                | `true`    | Enable IPv4                          |
| `avahi_use_ipv6`                | `true`    | Enable IPv6                          |
| `avahi_allow_interfaces`        | `[]`      | Restrict to interfaces (empty = all) |
| `avahi_deny_interfaces`         | `[]`      | Deny specific interfaces             |
| `avahi_cache_entries_max`       | `4096`    | Max DNS cache entries per interface  |
| `avahi_ratelimit_interval_usec` | `1000000` | Rate limit interval (microseconds)   |
| `avahi_ratelimit_burst`         | `1000`    | Rate limit burst count               |

### D-Bus

| Variable            | Default | Description            |
|---------------------|---------|------------------------|
| `avahi_enable_dbus` | `true`  | Enable D-Bus interface |

### Wide Area

| Variable                 | Default | Description             |
|--------------------------|---------|-------------------------|
| `avahi_enable_wide_area` | `false` | DNS-SD over unicast DNS |

### Publish Settings

| Variable                                | Default | Description                         |
|-----------------------------------------|---------|-------------------------------------|
| `avahi_disable_publishing`              | `false` | Disable all publishing (query-only) |
| `avahi_disable_user_service_publishing` | `false` | Block user service publishing       |
| `avahi_publish_addresses`               | `true`  | Publish IP address records          |
| `avahi_publish_hinfo`                   | `false` | Publish HINFO (privacy concern)     |
| `avahi_publish_workstation`             | `false` | Publish _workstation._tcp           |
| `avahi_publish_domain`                  | `true`  | Announce domain for browsing        |
| `avahi_publish_aaaa_on_ipv4`            | `true`  | Publish AAAA via IPv4               |
| `avahi_publish_a_on_ipv6`               | `false` | Publish A via IPv6                  |

### Reflector

| Variable                 | Default | Description                     |
|--------------------------|---------|---------------------------------|
| `avahi_enable_reflector` | `false` | Forward mDNS between interfaces |
| `avahi_reflect_ipv`      | `false` | Forward between IPv4/IPv6       |

### Resource Limits

| Variable              | Default   | Description                |
|-----------------------|-----------|----------------------------|
| `avahi_rlimit_core`   | `0`       | Core dump size limit       |
| `avahi_rlimit_data`   | `8388608` | Data segment size limit    |
| `avahi_rlimit_fsize`  | `0`       | File creation size limit   |
| `avahi_rlimit_nofile` | `768`     | Open file descriptor limit |
| `avahi_rlimit_stack`  | `8388608` | Stack size limit           |

### NSS-mDNS

| Variable                 | Default          | Description                           |
|--------------------------|------------------|---------------------------------------|
| `avahi_nss_mdns_enabled` | `true`           | Enable nss-mdns for .local resolution |
| `avahi_nsswitch_hosts`   | *(see defaults)* | Full nsswitch.conf hosts line         |

### Firewall

| Variable                  | Default    | Description                  |
|---------------------------|------------|------------------------------|
| `avahi_firewalld_enabled` | `false`    | Enable firewalld integration |
| `avahi_firewalld_zone`    | `'public'` | Zone for mDNS service        |

### AppArmor

| Variable                 | Default | Description                         |
|--------------------------|---------|-------------------------------------|
| `avahi_apparmor_enabled` | `false` | Enable AppArmor profile enforcement |

## Tags

| Tag               | Scope                            |
|-------------------|----------------------------------|
| `avahi`           | All role tasks                   |
| `avahi:install`   | Package installation             |
| `avahi:configure` | Configuration files and nsswitch |
| `avahi:service`   | Service management               |
| `avahi:firewall`  | Firewalld rules                  |
| `avahi:apparmor`  | AppArmor profile enforcement     |

## Example Playbook

```yaml
- name: Include avahi role
  ansible.builtin.include_role:
    name: marcstraube.common.avahi
  tags:
    - avahi
  when: avahi_enabled | default(true) | bool
```

```yaml
- name: Include avahi role
  ansible.builtin.include_role:
    name: marcstraube.common.avahi
  tags:
    - avahi
  when: avahi_enabled | default(true) | bool
  vars:
    avahi_publish_workstation: true
    avahi_firewalld_enabled: true
    avahi_enable_reflector: true
```

## Testing

```bash
cd roles/avahi
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

- **nsswitch.conf**: The role replaces the entire `hosts:` line. Customize
  `avahi_nsswitch_hosts` for environments with SSSD, Winbind, or other NSS modules.
- **Firewall**: mDNS uses UDP 5353 (multicast). The firewalld `mdns` service is
  predefined. Enable with `avahi_firewalld_enabled: true`.
- **AppArmor**: Not applicable on RHEL/Rocky (uses SELinux). The profile
  `usr.sbin.avahi-daemon` is shipped by the AppArmor packages, not by Avahi itself.
- **`rlimit-nproc`**: Intentionally omitted from the config template. Upstream removed
  the default due to container UID namespace sharing issues.

## License

MIT

## Author

Marc Straube
