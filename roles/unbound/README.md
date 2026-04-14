# marcstraube.common.unbound

Configures [Unbound](https://nlnetlabs.nl/projects/unbound/about/) as a local DNS resolver
with DNSSEC validation, DNS-over-TLS upstream, ad/tracker blocklists, and local zone
management.

## Description

- DNSSEC validation with automatic root trust anchor management
- DNS-over-TLS upstream forwarding
- Ad/tracker blocking via configurable blocklist URLs (systemd timer)
- Local zones and records (A, AAAA, PTR)
- Forward zones with TLS support
- Root hints auto-update (weekly systemd timer)
- Remote control via unbound-control (Unix socket)
- Firewalld integration (DNS port 53)
- AppArmor profile enforcement (Debian ships profile)

## Requirements

- `ansible.posix` collection (for firewalld tasks)

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

See [`defaults/main.yml`](defaults/main.yml) for all variables with descriptions.

### Role Control

| Variable                  | Default | Description                          |
|---------------------------|---------|--------------------------------------|
| `unbound_enabled`         | `true`  | Enable the unbound role              |
| `unbound_service_enabled` | `true`  | Enable and start the unbound service |

### Key Variables

| Variable                              | Default                 | Description                            |
|---------------------------------------|-------------------------|----------------------------------------|
| `unbound_interfaces`                  | `['127.0.0.1', '::1']`  | Interfaces to listen on                |
| `unbound_port`                        | `53`                    | Listen port                            |
| `unbound_dnssec_enabled`              | `true`                  | Enable DNSSEC validation               |
| `unbound_tls_upstream`                | `true`                  | Configure TLS cert bundle for upstream |
| `unbound_forward_zones`               | `[]`                    | List of forward zone definitions       |
| `unbound_local_zones`                 | `[]`                    | Local zone definitions                 |
| `unbound_local_data`                  | `[]`                    | Simple local A/AAAA records            |
| `unbound_blocklist_enabled`           | `false`                 | Enable ad/tracker blocking             |
| `unbound_root_hints_update_enabled`   | `true`                  | Auto-update root hints weekly          |
| `unbound_remote_control_enabled`      | `true`                  | Enable unbound-control socket          |

### Firewall

| Variable                    | Default    | Description                      |
|-----------------------------|------------|----------------------------------|
| `unbound_firewalld_enabled` | `false`    | Enable firewalld integration     |
| `unbound_firewalld_zone`    | `'public'` | Firewalld zone for service rules |

### AppArmor

| Variable                   | Default | Description                         |
|----------------------------|---------|-------------------------------------|
| `unbound_apparmor_enabled` | `false` | Enable AppArmor profile enforcement |

## Tags

| Tag                 | Scope                                                  |
|---------------------|--------------------------------------------------------|
| `unbound`           | All role tasks                                         |
| `unbound:install`   | Package installation, root hints download, timer setup |
| `unbound:configure` | Config files, DNSSEC anchor, unbound-control keys      |
| `unbound:blocklist` | Blocklist script, timer, initial download              |
| `unbound:service`   | Service management                                     |
| `unbound:firewall`  | Firewalld DNS rule                                     |
| `unbound:apparmor`  | AppArmor profile enforcement                           |

## Example Playbook

```yaml
- name: Configure DNS resolver
  hosts: dns_servers
  become: true
  roles:
    - role: marcstraube.common.unbound
      vars:
        unbound_interfaces:
          - '0.0.0.0'
          - '::0'
        unbound_access_control:
          - subnet: '127.0.0.0/8'
            action: 'allow'
          - subnet: '10.0.0.0/8'
            action: 'allow'
        unbound_forward_zones:
          - name: '.'
            forward_addresses:
              - '1.1.1.1@853#cloudflare-dns.com'
              - '9.9.9.9@853#dns.quad9.net'
            forward_tls_upstream: true
        unbound_blocklist_enabled: true
        unbound_firewalld_enabled: true
```

## Testing

```bash
cd roles/unbound
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

- On BTRFS filesystems, NoCOW (`chattr +C`) is applied to `/var/lib/unbound/` to avoid
  copy-on-write overhead for DNS cache and root hints data.
- Rocky 9 ships unbound 1.16.2 which is significantly older than other platforms. All
  directives used in this role are compatible with 1.16+.
- On Rocky 10, `unbound-anchor` is a separate package (automatically installed).
- Debian ships an AppArmor profile for unbound (enforced by default when AppArmor is active).
- The blocklist feature downloads hosts-format blocklists and converts them to unbound
  `local-zone` entries.

## License

MIT

## Author

Marc Straube
