# marcstraube.common.resolved

Manage systemd-resolved DNS resolver.

## Description

Installs and configures systemd-resolved as the system DNS resolver on all
supported platforms. Manages `/etc/systemd/resolved.conf` with configurable
DNS servers, DNSSEC, DNS-over-TLS, multicast DNS, caching, and stub listener
settings. Optionally manages the `/etc/resolv.conf` symlink to the resolved
stub resolver.

## Requirements

- ansible-core >= 2.17
- No additional collections required

## Supported Platforms

| Platform                   | systemd | Notes                         |
|----------------------------|---------|-------------------------------|
| Arch Linux                 | 260     |                               |
| Debian Trixie              | 257     |                               |
| EL 9 (Rocky, Alma, RHEL)   | 252     | Technology Preview on RHEL 9  |
| EL 10 (Rocky, Alma, RHEL)  | 257     | Fully supported since RHEL 10 |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable                     | Default | Description                                      |
|------------------------------|---------|--------------------------------------------------|
| `resolved_enabled`           | `true`  | Enable the resolved role                         |
| `resolved_service_enabled`   | `true`  | Enable and start the systemd-resolved service    |

### DNS Servers

| Variable                  | Default | Description                                              |
|---------------------------|---------|----------------------------------------------------------|
| `resolved_dns`            | `[]`    | DNS server addresses (supports port and SNI)             |
| `resolved_fallback_dns`   | `[]`    | Fallback DNS servers                                     |
| `resolved_domains`        | `[]`    | Search and routing domains (`~` prefix for routing-only) |

### Resolution Protocols

| Variable                  | Default | Description                                                     |
|---------------------------|---------|-----------------------------------------------------------------|
| `resolved_llmnr`          | `''`    | Link-Local Multicast Name Resolution (`true`/`false`/`resolve`) |
| `resolved_multicast_dns`  | `''`    | Multicast DNS (`true`/`false`/`resolve`)                        |
| `resolved_dnssec`         | `''`    | DNSSEC validation (`true`/`false`/`allow-downgrade`)            |
| `resolved_dns_over_tls`   | `''`    | DNS-over-TLS (`true`/`false`/`opportunistic`)                   |

### Cache

| Variable                         | Default | Description                                              |
|----------------------------------|---------|----------------------------------------------------------|
| `resolved_cache`                 | `''`    | DNS response caching (`true`/`false`/`no-negative`)      |
| `resolved_cache_from_localhost`  | `''`    | Cache responses from localhost                           |
| `resolved_stale_retention_sec`   | `''`    | Stale cache retention in seconds (systemd 254+, not EL9) |

### Stub Listener

| Variable                         | Default | Description                                     |
|----------------------------------|---------|-------------------------------------------------|
| `resolved_stub_listener`         | `''`    | DNS stub listener (`true`/`false`/`udp`/`tcp`)  |
| `resolved_stub_listener_extra`   | `[]`    | Additional stub listener addresses              |

### Miscellaneous

| Variable                                  | Default | Description                                |
|-------------------------------------------|---------|--------------------------------------------|
| `resolved_read_etc_hosts`                 | `''`    | Read /etc/hosts for name resolution        |
| `resolved_resolve_unicast_single_label`   | `''`    | Forward single-label names to unicast DNS  |

### resolv.conf

| Variable                      | Default | Description                                      |
|-------------------------------|---------|--------------------------------------------------|
| `resolved_manage_resolv_conf` | `true`  | Manage /etc/resolv.conf symlink to stub resolver |

Empty string defaults (`''`) mean the option is omitted from the configuration
file, leaving the systemd compiled-in default in place.

## Tags

| Tag                  | Scope                              |
|----------------------|------------------------------------|
| `resolved`           | All role tasks                     |
| `resolved:install`   | Package installation               |
| `resolved:configure` | Configuration                      |
| `resolved:service`   | Service and resolv.conf management |

## Integration with NetworkManager and WireGuard

This role is designed to work together with the `networkmanager` and
`wireguard` roles to provide a complete DNS resolution chain:

```text
DHCP/static DNS ──► NetworkManager ──D-Bus──► systemd-resolved ◄──resolvconf── wg-quick
                                                    │
                                              127.0.0.53:53
                                                    │
                                        /etc/resolv.conf (symlink)
                                                    │
                                              applications
```

### Setup

1. **resolved role** (runs first): installs, configures, and starts the
   resolver; creates the resolv.conf symlink
2. **networkmanager role**: set `networkmanager_dns: 'systemd-resolved'`
   so NM pushes DHCP-obtained DNS servers to resolved via D-Bus
3. **wireguard role**: `dns:` entries in interface config are pushed to
   resolved per-interface via `resolvconf` when the tunnel comes up

### Example Inventory (group_vars/all)

```yaml
# Enable systemd-resolved
resolved_enabled: true
resolved_dnssec: 'allow-downgrade'
resolved_dns_over_tls: 'opportunistic'

# NetworkManager uses resolved as DNS backend
networkmanager_dns: 'systemd-resolved'

# WireGuard client with tunnel DNS
wireguard_interfaces:
  - name: wg0
    mode: client
    dns:
      - '10.100.0.1'
    # ...
```

### Playbook Ordering

In `base_system.yml`, the resolved role runs **before** networkmanager to
ensure the resolver is active when NM starts pushing DNS information:

```yaml
- name: Base System | Include resolved role
  ansible.builtin.include_role:
    name: marcstraube.common.resolved
  # ...

- name: Base System | Include networkmanager role
  ansible.builtin.include_role:
    name: marcstraube.common.networkmanager
  # ...
```

The wireguard role runs between resolved and networkmanager in
`base_system.yml`, ensuring configs are deployed before NM dispatcher
scripts reference them.

## Example Playbook

```yaml
- name: Include resolved role
  ansible.builtin.include_role:
    name: marcstraube.common.resolved
  tags:
    - resolved
  when: resolved_enabled | default(true) | bool
```

## Testing

```bash
cd roles/resolved
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

- On Arch Linux, systemd-resolved is part of the base `systemd` package. This
  role installs `systemd-resolvconf` which provides the `resolvconf`
  compatibility wrapper needed by wg-quick and other DHCP/VPN clients.
- On Debian and RedHat, the `systemd-resolved` package is separate and
  includes the resolvconf wrapper.
- On RHEL 9, systemd-resolved is a Technology Preview. It works but is not
  covered by Red Hat production support SLAs.
- DNSSEC validation is a Technology Preview on RHEL 10 — consider using
  `allow-downgrade` rather than `true` on EL systems.
- `resolved_stale_retention_sec` requires systemd 254+. On EL9 (systemd 252),
  the option is silently ignored with a journal warning.

## References

- [resolved.conf(5) man page](https://www.freedesktop.org/software/systemd/man/latest/resolved.conf.html)
- [systemd-resolved(8) man page](https://www.freedesktop.org/software/systemd/man/latest/systemd-resolved.service.html)
- [NetworkManager DNS plugin documentation](https://networkmanager.dev/docs/api/latest/NetworkManager.conf.html)
- [RHEL 10 — systemd-resolved support status](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/)

## License

MIT

## Author

Marc Straube
