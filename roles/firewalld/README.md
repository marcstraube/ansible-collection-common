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

| Platform | firewalld Version | Notes |
| -------- | ----------------- | ----- |
| Arch Linux | 2.x | nftables backend, full feature set |
| Debian Trixie | 2.x | nftables backend, full feature set |
| Rocky Linux 9 | 1.x | Some 2.x features ignored (strict_forward_ports, flowtable, counters, table_owner) |
| Rocky Linux 10 | 2.x | nftables backend, full feature set |

## Role Variables

### Service Control

| Variable            | Default | Description                      |
| ------------------- | ------- | -------------------------------- |
| `firewalld_enabled` | `true`  | Enable/disable firewalld service |

### Daemon Settings

Deployed as `/etc/firewalld/firewalld.conf`.

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `firewalld_default_zone` | `public` | Default zone for unassigned interfaces |
| `firewalld_backend` | `''` | Backend: `nftables` or `iptables` (empty = system default) |
| `firewalld_log_denied` | `off` | Log denied packets: `off`, `all`, `unicast`, `broadcast`, `multicast` |
| `firewalld_cleanup_on_exit` | `true` | Clean up rules on daemon exit |
| `firewalld_cleanup_modules_on_exit` | `false` | Clean up kernel modules on exit |
| `firewalld_lockdown` | `false` | Restrict firewall modification to whitelisted apps |
| `firewalld_ipv6_rpfilter` | `strict` | IPv6 reverse path filter |
| `firewalld_individual_calls` | `false` | Use individual calls instead of transactions |
| `firewalld_flush_all_on_reload` | `true` | Flush all rules on reload |
| `firewalld_reload_policy` | `INPUT:DROP,FORWARD:DROP,OUTPUT:DROP` | Default policy during reload |
| `firewalld_rfc3964_ipv4` | `true` | Auto-add 6to4/Teredo rules |
| `firewalld_strict_forward_ports` | `false` | Require forwarding targets in same zone (2.x only) |
| `firewalld_nftables_flowtable` | `off` | nftables flowtable offloading (2.x only) |
| `firewalld_nftables_counters` | `false` | nftables packet counters (2.x only) |
| `firewalld_nftables_table_owner` | `true` | nftables table owner flag (2.x only) |

### Zone Management

| Variable          | Default | Description                                  |
| ----------------- | ------- | -------------------------------------------- |
| `firewalld_zones` | `[]`    | Zones to create or modify (`{name, target}`) |

### Services

| Variable                      | Default   | Description                                     |
| ----------------------------- | --------- | ----------------------------------------------- |
| `firewalld_services`          | `['ssh']` | Services to enable in default zone              |
| `firewalld_zone_services`     | `[]`      | Services per zone (`{zone, services}`)          |
| `firewalld_services_disabled` | `[]`      | Services to explicitly disable                  |
| `firewalld_purge_services`    | `false`   | Remove unconfigured services from managed zones |

### Ports

| Variable               | Default | Description                                     |
| ---------------------- | ------- | ----------------------------------------------- |
| `firewalld_ports`      | `[]`    | Ports to open in default zone (`port/protocol`) |
| `firewalld_zone_ports` | `[]`    | Ports per zone (`{zone, ports}`)                |

### Rich Rules

| Variable               | Default | Description                           |
| ---------------------- | ------- | ------------------------------------- |
| `firewalld_rich_rules` | `[]`    | Rich rules for complex firewall logic |

### Source-Based Routing

| Variable            | Default | Description                                          |
| ------------------- | ------- | ---------------------------------------------------- |
| `firewalld_sources` | `[]`    | Source networks assigned to zones (`{zone, source}`) |

### Interface Assignment

| Variable               | Default | Description                                        |
| ---------------------- | ------- | -------------------------------------------------- |
| `firewalld_interfaces` | `[]`    | Interfaces assigned to zones (`{zone, interface}`) |

### Port Forwarding

| Variable             | Default | Description                                        |
| -------------------- | ------- | -------------------------------------------------- |
| `firewalld_forwards` | `[]`    | Port forwarding rules (`{port, to_port, to_addr}`) |

### Masquerading (NAT)

| Variable               | Default | Description                                      |
| ---------------------- | ------- | ------------------------------------------------ |
| `firewalld_masquerade` | `[]`    | Enable masquerading per zone (`{zone, enabled}`) |

### ICMP Blocking

| Variable                | Default | Description                  |
| ----------------------- | ------- | ---------------------------- |
| `firewalld_icmp_blocks` | `[]`    | ICMP types to block per zone |

### Direct Rules

| Variable                 | Default | Description                                           |
| ------------------------ | ------- | ----------------------------------------------------- |
| `firewalld_direct_rules` | `[]`    | Low-level nftables/iptables rules (deprecated in 2.x) |

### Custom Services

| Variable                    | Default | Description                    |
| --------------------------- | ------- | ------------------------------ |
| `firewalld_custom_services` | `[]`    | Custom service XML definitions |

## Tags

| Tag                   | Scope                       |
| --------------------- | --------------------------- |
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

## License

MIT

## Author

Marc Straube
