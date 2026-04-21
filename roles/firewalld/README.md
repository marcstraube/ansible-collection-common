# marcstraube.common.firewalld

Configure firewalld zones, services, ports, and rich rules.

## Description

Installs and configures firewalld as the host firewall. Manages daemon settings,
zones, services, ports, rich rules, source-based routing, interface assignments,
port forwarding, masquerading, ICMP blocking, direct rules, and custom service
definitions. This is the infrastructure-wide standard firewall role.

## Requirements

- ansible-core >= 2.17
- Collections: `ansible.posix`

## Supported Platforms

| Platform                  | Notes                                |
|---------------------------|--------------------------------------|
| Arch Linux                | firewalld 2.x, full feature set      |
| Debian Trixie             | firewalld 2.x, full feature set      |
| EL 9 (Rocky, Alma, RHEL)  | firewalld 1.x, some 2.x features N/A |
| EL 10 (Rocky, Alma, RHEL) | firewalld 2.x, full feature set      |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable                    | Default | Description                            |
|-----------------------------|---------|----------------------------------------|
| `firewalld_enabled`         | `true`  | Enable the firewalld role              |
| `firewalld_service_enabled` | `true`  | Enable and start the firewalld service |

### Daemon Settings

Deployed as `/etc/firewalld/firewalld.conf`.

| Variable                            | Default  | Description                        |
|-------------------------------------|----------|------------------------------------|
| `firewalld_default_zone`            | `public` | Default zone                       |
| `firewalld_backend`                 | `''`     | `nftables` or `iptables`           |
| `firewalld_log_denied`              | `off`    | Log denied packets                 |
| `firewalld_cleanup_on_exit`         | `true`   | Clean rules on exit                |
| `firewalld_cleanup_modules_on_exit` | `false`  | Clean modules on exit              |
| `firewalld_lockdown`                | `false`  | Restrict to whitelisted apps       |
| `firewalld_ipv6_rpfilter`           | `strict` | IPv6 reverse path filter           |
| `firewalld_individual_calls`        | `false`  | Individual calls (no transactions) |
| `firewalld_flush_all_on_reload`     | `true`   | Flush all rules on reload          |
| `firewalld_reload_policy`           | `*:DROP` | Policy during reload               |
| `firewalld_rfc3964_ipv4`            | `true`   | Auto-add 6to4/Teredo rules         |
| `firewalld_strict_forward_ports`    | `false`  | Same-zone forwarding only (2.x)    |
| `firewalld_nftables_flowtable`      | `off`    | Flowtable offloading (2.x)         |
| `firewalld_nftables_counters`       | `false`  | Packet counters (2.x)              |
| `firewalld_nftables_table_owner`    | `true`   | Table owner flag (2.x)             |

### Zone Management

| Variable          | Default | Description                                  |
|-------------------|---------|----------------------------------------------|
| `firewalld_zones` | `[]`    | Zones to create or modify (`{name, target}`) |

### Services

| Variable                      | Default   | Description                                     |
|-------------------------------|-----------|-------------------------------------------------|
| `firewalld_services`          | `['ssh']` | Services to enable in default zone              |
| `firewalld_zone_services`     | `[]`      | Services per zone (`{zone, services}`)          |
| `firewalld_services_disabled` | `[]`      | Services to explicitly disable                  |
| `firewalld_purge_services`    | `false`   | Remove unconfigured services from managed zones |

### Ports

| Variable               | Default | Description                                     |
|------------------------|---------|-------------------------------------------------|
| `firewalld_ports`      | `[]`    | Ports to open in default zone (`port/protocol`) |
| `firewalld_zone_ports` | `[]`    | Ports per zone (`{zone, ports}`)                |

### Rich Rules

| Variable               | Default | Description                           |
|------------------------|---------|---------------------------------------|
| `firewalld_rich_rules` | `[]`    | Rich rules for complex firewall logic |

### Source-Based Routing

| Variable            | Default | Description                                          |
|---------------------|---------|------------------------------------------------------|
| `firewalld_sources` | `[]`    | Source networks assigned to zones (`{zone, source}`) |

### Interface Assignment

| Variable               | Default | Description                                        |
|------------------------|---------|----------------------------------------------------|
| `firewalld_interfaces` | `[]`    | Interfaces assigned to zones (`{zone, interface}`) |

### Port Forwarding

| Variable             | Default | Description                                        |
|----------------------|---------|----------------------------------------------------|
| `firewalld_forwards` | `[]`    | Port forwarding rules (`{port, to_port, to_addr}`) |

### Masquerading (NAT)

| Variable               | Default | Description                                      |
|------------------------|---------|--------------------------------------------------|
| `firewalld_masquerade` | `[]`    | Enable masquerading per zone (`{zone, enabled}`) |

### ICMP Blocking

| Variable                | Default | Description                  |
|-------------------------|---------|------------------------------|
| `firewalld_icmp_blocks` | `[]`    | ICMP types to block per zone |

### Direct Rules

| Variable                 | Default | Description                                           |
|--------------------------|---------|-------------------------------------------------------|
| `firewalld_direct_rules` | `[]`    | Low-level nftables/iptables rules (deprecated in 2.x) |

### Custom Services

| Variable                    | Default | Description                    |
|-----------------------------|---------|--------------------------------|
| `firewalld_custom_services` | `[]`    | Custom service XML definitions |

## Tags

| Tag                   | Scope                       |
|-----------------------|-----------------------------|
| `firewalld`           | All role tasks              |
| `firewalld:install`   | Package installation        |
| `firewalld:configure` | Configuration and rules     |
| `firewalld:purge`     | Purge unconfigured services |
| `firewalld:service`   | Service management          |

## Example Playbook

```yaml
- name: Configure firewalld
  hosts: all
  become: true
  tasks:
    - name: Include firewalld role
      ansible.builtin.include_role:
        name: marcstraube.common.firewalld
      tags: [firewalld]
      when: firewalld_enabled | default(true) | bool
```

### Web Server with Custom Zone

```yaml
- name: Include firewalld role
  ansible.builtin.include_role:
    name: marcstraube.common.firewalld
  vars:
    firewalld_services:
      - 'ssh'
      - 'http'
      - 'https'
    firewalld_sources:
      - zone: 'trusted'
        source: '10.0.0.0/8'
    firewalld_purge_services: true
```

## Testing

```bash
cd roles/firewalld
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## References

- [firewalld](https://firewalld.org/) — Dynamically managed firewall with network zones
- [firewalld Documentation](https://firewalld.org/documentation/) — Official documentation

## License

MIT

## Author

Marc Straube
