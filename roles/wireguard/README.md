# marcstraube.common.wireguard

Configure WireGuard VPN tunnels with multi-interface support.

## Description

Manages one or more WireGuard VPN interfaces per host, each in server or client
mode. Features automatic key generation, peer management, lifecycle scripts
(pre/post up/down), IP forwarding via sysctl, firewalld integration, and systemd
service management via wg-quick.

Each interface is independently configured and managed as a
`wg-quick@<name>.service` systemd unit.

## Requirements

- ansible-core >= 2.17
- `ansible.posix` collection (firewalld, sysctl)
- `community.general` collection (modprobe)

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

| Variable                    | Default            | Description                             |
|-----------------------------|--------------------|-----------------------------------------|
| `wireguard_enabled`         | `true`             | Enable the wireguard role               |
| `wireguard_service_enabled` | `true`             | Enable and start services per interface |
| `wireguard_config_dir`      | `'/etc/wireguard'` | Key/config directory                    |

### Interface Configuration

All interface settings are defined as a list of dicts in `wireguard_interfaces`.

```yaml
wireguard_interfaces:
  - name: wg0
    address: '10.100.0.1/24'
    port: 51820
    mode: server
    ip_forward: true
    peers:
      - name: laptop
        public_key: '{{ vault_wireguard_keys.wg0.peer_laptop_pubkey }}'
        allowed_ips: '10.100.0.2/32'
        persistent_keepalive: 25
```

#### Per-Interface Keys

| Key                  | Default      | Description                                      |
|----------------------|--------------|--------------------------------------------------|
| `name`               | **required** | Interface name (wg0, wg1, ...)                   |
| `address`            | **required** | IPv4 address (CIDR notation)                     |
| `address_ipv6`       | `''`         | IPv6 address (CIDR, optional)                    |
| `port`               | `51820`      | Listen port (UDP)                                |
| `private_key`        | `''`         | Private key (auto-generated if empty)            |
| `mode`               | `'server'`   | Interface mode: `server` or `client`             |
| `ip_forward`         | `true`       | Enable IP forwarding (server mode)               |
| `dns`                | `[]`         | DNS servers written to wg.conf (client mode)     |
| `peers`              | `[]`         | List of peer definitions (see below)             |
| `firewalld_enabled`  | `true`       | Enable firewalld integration                     |
| `firewalld_zone`     | `'public'`   | Firewalld zone for WireGuard port                |
| `preup`              | `[]`         | Commands run before interface up                 |
| `postup`             | `[]`         | Commands run after interface up                  |
| `predown`            | `[]`         | Commands run before interface down               |
| `postdown`           | `[]`         | Commands run after interface down                |
| `generate_keys`      | `true`       | Auto-generate keys if private_key is empty       |
| `mtu`                | `0`          | Interface MTU (0 = auto)                         |
| `table`              | `'auto'`     | Routing table (auto, off, or number)             |
| `fwmark`             | `''`         | Firewall mark for outgoing packets               |
| `saveconfig`         | `false`      | Save runtime config on shutdown                  |
| `config_mode`        | `'0600'`     | Config file permissions                          |

Use `%i` as placeholder for the interface name in lifecycle scripts.

#### Peer Definition

| Key                    | Default | Description                               |
|------------------------|---------|-------------------------------------------|
| `name`                 |         | Peer name (used as comment in config)     |
| `public_key`           |         | Peer's public key                         |
| `allowed_ips`          |         | Allowed IPs for this peer                 |
| `preshared_key`        | `''`    | Preshared key for additional security     |
| `endpoint`             | `''`    | Endpoint address (IP:Port)                |
| `persistent_keepalive` | `0`     | Keepalive interval (seconds, 0 = off)     |
| `description`          | `''`    | Description/comment                       |

### Server Mode Example

```yaml
wireguard_interfaces:
  - name: wg0
    address: '10.100.0.1/24'
    port: 51820
    private_key: '{{ vault_wireguard_keys.wg0.private_key }}'
    mode: server
    ip_forward: true
    peers:
      - name: laptop
        public_key: '{{ vault_wireguard_keys.wg0.peer_laptop_pubkey }}'
        allowed_ips: '10.100.0.2/32'
        persistent_keepalive: 25
      - name: phone
        public_key: '{{ vault_wireguard_keys.wg0.peer_phone_pubkey }}'
        allowed_ips: '10.100.0.3/32'
```

### Client Mode Example

```yaml
wireguard_interfaces:
  - name: wg0
    address: '10.100.0.2/32'
    port: 51820
    private_key: '{{ vault_wireguard_keys.wg0.private_key }}'
    mode: client
    dns:
      - '10.100.0.1'
    peers:
      - name: server
        public_key: '{{ vault_wireguard_keys.wg0.server_pubkey }}'
        endpoint: '{{ vault_wireguard_keys.wg0.server_endpoint }}'
        allowed_ips: '10.100.0.0/24'
        persistent_keepalive: 25
```

### Multi-Interface Example

```yaml
wireguard_interfaces:
  - name: wg0
    address: '10.100.0.1/24'
    port: 51820
    mode: server
    ip_forward: true
    peers:
      - name: road-warrior
        public_key: '{{ vault_wireguard_keys.wg0.peer_rw_pubkey }}'
        allowed_ips: '10.100.0.2/32'
  - name: wg1
    address: '10.200.0.2/32'
    port: 51821
    mode: client
    dns:
      - '10.200.0.1'
    firewalld_enabled: false
    peers:
      - name: corporate-vpn
        public_key: '{{ vault_wireguard_keys.wg1.server_pubkey }}'
        endpoint: '{{ vault_wireguard_keys.wg1.server_endpoint }}'
        allowed_ips: '10.200.0.0/16'
        persistent_keepalive: 25
```

## Tags

| Tag                   | Scope                                       |
|-----------------------|---------------------------------------------|
| `wireguard`           | All tasks                                   |
| `wireguard:install`   | Package installation, kernel module, sysctl |
| `wireguard:keys`      | Key generation and management               |
| `wireguard:configure` | Configuration template deployment           |
| `wireguard:service`   | Service management                          |
| `wireguard:firewall`  | Firewalld rules                             |

## Example Playbook

```yaml
- name: Include wireguard role
  ansible.builtin.include_role:
    name: marcstraube.common.wireguard
  tags:
    - wireguard
  when: wireguard_enabled | default(true) | bool
```

## Testing

```bash
cd roles/wireguard
molecule test
```

Driver: `vagrant` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## References

- [WireGuard](https://www.wireguard.com/) — fast, modern, secure VPN tunnel
- [WireGuard Quickstart](https://www.wireguard.com/quickstart/) — installation and setup guide

## License

MIT

## Author

Marc Straube
