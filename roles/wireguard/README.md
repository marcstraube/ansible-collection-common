# marcstraube.common.wireguard

Configure WireGuard VPN tunnels in server or client mode.

## Description

Manages WireGuard VPN interface configuration in server or client mode with
automatic key generation, peer management, lifecycle scripts (pre/post up/down),
IP forwarding via sysctl, firewalld integration, and systemd service management.

## Requirements

- ansible-core >= 2.17
- `ansible.posix` collection (firewalld, sysctl)
- `community.general` collection (modprobe)

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

| Variable                    | Default | Description                            |
|-----------------------------|---------|----------------------------------------|
| `wireguard_enabled`         | `true`  | Enable the wireguard role              |
| `wireguard_service_enabled` | `true`  | Enable and start the WireGuard service |

### Interface Configuration

| Variable                 | Default           | Description                           |
|--------------------------|-------------------|---------------------------------------|
| `wireguard_interface`    | `'wg0'`           | Interface name                        |
| `wireguard_address`      | `'10.100.0.1/24'` | IPv4 address (CIDR)                   |
| `wireguard_address_ipv6` | `''`              | IPv6 address (CIDR, optional)         |
| `wireguard_port`         | `51820`           | Listen port (UDP)                     |
| `wireguard_private_key`  | `''`              | Private key (auto-generated if empty) |
| `wireguard_dns`          | `['10.100.0.1']`  | DNS servers (client mode)             |
| `wireguard_mtu`          | `0`               | Interface MTU (0 = auto)              |
| `wireguard_table`        | `'auto'`          | Routing table (auto, off, or number)  |
| `wireguard_fwmark`       | `''`              | Firewall mark for outgoing packets    |
| `wireguard_saveconfig`   | `false`           | Save runtime config on shutdown       |

### Server Mode

| Variable                | Default | Description                 |
|-------------------------|---------|-----------------------------|
| `wireguard_server_mode` | `true`  | Enable server mode          |
| `wireguard_ip_forward`  | `true`  | Enable IPv4/IPv6 forwarding |

### Peers

| Variable          | Default | Description              |
|-------------------|---------|--------------------------|
| `wireguard_peers` | `[]`    | List of peer definitions |

Each peer dict supports: `name`, `public_key`, `allowed_ips`, `preshared_key`,
`endpoint`, `persistent_keepalive`, `description`.

### Client Mode

| Variable                                | Default           | Description                  |
|-----------------------------------------|-------------------|------------------------------|
| `wireguard_client_mode`                 | `false`           | Enable client mode           |
| `wireguard_client_endpoint`             | `''`              | Server endpoint (IP:Port)    |
| `wireguard_client_server_pubkey`        | `''`              | Server public key            |
| `wireguard_client_preshared_key`        | `''`              | Preshared key (optional)     |
| `wireguard_client_allowed_ips`          | `'10.100.0.0/24'` | Allowed IPs to route         |
| `wireguard_client_persistent_keepalive` | `25`              | Keepalive interval (seconds) |

### Lifecycle Scripts

| Variable             | Default | Description                        |
|----------------------|---------|------------------------------------|
| `wireguard_preup`    | `[]`    | Commands run before interface up   |
| `wireguard_postup`   | `[]`    | Commands run after interface up    |
| `wireguard_predown`  | `[]`    | Commands run before interface down |
| `wireguard_postdown` | `[]`    | Commands run after interface down  |

Use `%i` as placeholder for the interface name.

### Firewall

| Variable                      | Default    | Description                       |
|-------------------------------|------------|-----------------------------------|
| `wireguard_firewalld_enabled` | `true`     | Enable firewalld integration      |
| `wireguard_firewalld_zone`    | `'public'` | Firewalld zone for WireGuard port |

### Key Management

| Variable                  | Default            | Description                  |
|---------------------------|--------------------|------------------------------|
| `wireguard_config_dir`    | `'/etc/wireguard'` | Key/config directory         |
| `wireguard_generate_keys` | `true`             | Auto-generate WireGuard keys |
| `wireguard_config_mode`   | `'0600'`           | Config file permissions      |

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

## License

MIT

## Author

Marc Straube
