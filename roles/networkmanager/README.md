# marcstraube.common.networkmanager

Manage NetworkManager daemon configuration, connections, and dispatcher scripts.

## Description

This role provides comprehensive NetworkManager management:

- **Daemon configuration** via `/etc/NetworkManager/conf.d/00-ansible.conf` (DNS backend,
  logging, connectivity checking, WiFi power saving, MAC randomization, etc.)
- **Connection profiles** (Ethernet, WiFi, WireGuard, VPN) via `community.general.nmcli`
- **Dispatcher scripts** for automatic SMB/SSHFS mounting, VPN auto-connect, WiFi toggling,
  timezone detection
- **Polkit rules** for unprivileged NetworkManager management
- **Global DNS** applied to connections without explicit DNS config
- **Per-connection DHCP hostname control** (IPv4/IPv6 separately)

## Requirements

- **Ansible**: >= 2.17
- **Collections**: `community.general`

## Supported Platforms

| Platform                  | Notes                             |
|---------------------------|-----------------------------------|
| Arch Linux                | Full install + configure          |
| Debian Trixie             | Full install + configure          |
| EL 9 (Rocky, Alma, RHEL)  | Configure only (NM pre-installed) |
| EL 10 (Rocky, Alma, RHEL) | Configure only (NM pre-installed) |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

```yaml
networkmanager_enabled: true            # enable the networkmanager role
networkmanager_service_enabled: true    # enable and start the networkmanager service
```

### Daemon Configuration

Deployed to `/etc/NetworkManager/conf.d/00-ansible.conf`. Empty strings are omitted,
leaving distro defaults in place.

#### [main] section

```yaml
networkmanager_dns: ''                    # default, systemd-resolved, dnsmasq, none
networkmanager_rc_manager: ''             # auto, symlink, file, resolvconf, netconfig, unmanaged
networkmanager_systemd_resolved: true     # push DNS to systemd-resolved in addition to dns backend
networkmanager_auth_polkit: ''            # true, false, root-only
networkmanager_hostname_mode: ''          # default, dhcp, none
networkmanager_firewall_backend: ''       # iptables, nftables, none (auto-detected)
networkmanager_no_auto_default: []        # device specs excluded from auto wired connections
networkmanager_ignore_carrier: []         # device specs ignoring carrier state
networkmanager_plugins: []                # settings storage plugins (keyfile always appended)
networkmanager_migrate_ifcfg_rh: false    # auto-convert ifcfg-rh to keyfile (RHEL/Rocky)
```

#### [logging] section

```yaml
networkmanager_log_level: ''       # OFF, ERR, WARN, INFO, DEBUG, TRACE
networkmanager_log_domains: ''     # e.g. "WIFI:DEBUG,VPN:TRACE"
networkmanager_log_backend: ''     # syslog, journal
networkmanager_log_audit: false    # send audit events to auditd
```

#### [connectivity] section

```yaml
networkmanager_connectivity_uri: ''         # URL for probe (empty disables checking)
networkmanager_connectivity_interval: 300   # check frequency in seconds
networkmanager_connectivity_timeout: 20     # response timeout in seconds
networkmanager_connectivity_response: ''    # expected response body
```

#### [connection] section (global defaults)

```yaml
networkmanager_wifi_powersave: ''           # ignore, disable, enable
networkmanager_connection_mdns: ''          # 0=off, 1=resolve, 2=resolve+register
networkmanager_connection_llmnr: ''         # 0=off, 1=resolve, 2=resolve+register
networkmanager_ethernet_wake_on_lan: ''     # default, magic, unicast, etc.
networkmanager_dhcp_send_hostname_ipv4: true
networkmanager_dhcp_send_hostname_ipv6: true
```

#### [device] section

```yaml
networkmanager_wifi_scan_rand_mac: true    # MAC randomization during scanning
networkmanager_wifi_backend: ''            # wpa_supplicant, iwd
```

#### [ifupdown] section (Debian-only)

```yaml
networkmanager_ifupdown_managed: false     # manage /etc/network/interfaces devices
```

### Installation

```yaml
networkmanager_applet_enabled: false   # nm-applet + nm-connection-editor
networkmanager_vpn_plugins: []         # VPN plugin packages (OS-native names)
```

VPN plugin package names differ by OS:

| Plugin      | Arch                         | Debian                        | Rocky                        |
|-------------|------------------------------|-------------------------------|------------------------------|
| OpenVPN     | `networkmanager-openvpn`     | `network-manager-openvpn`     | `NetworkManager-openvpn`     |
| OpenConnect | `networkmanager-openconnect` | `network-manager-openconnect` | `NetworkManager-openconnect` |
| VPNC        | `networkmanager-vpnc`        | `network-manager-vpnc`        | `NetworkManager-vpnc`        |

### Connection Management

```yaml
networkmanager_connections:
  ethernet: {}
  wifi: {}
  wireguard: {}
  vpn: {}
```

#### Ethernet Connections

```yaml
networkmanager_connections:
  ethernet:
    "Office":
      ifname: "eth0"
      state: "present"
      autoconnect: true
      ip4: "192.168.1.10/24"
      gw4: "192.168.1.1"
      dns4: ["1.1.1.1", "8.8.8.8"]
      dns4_search: ["office.local"]
      routes4: ["10.0.0.0/8 192.168.1.254"]
      route_metric4: 100
      dhcp_send_hostname_ipv4: false    # per-connection override
      toggle_wifi: true                 # disable WiFi when connected
      vpn: ["Company VPN"]             # auto-connect VPN via connection.secondaries
```

#### WiFi Connections

Same options as Ethernet, plus `ssid`:

```yaml
networkmanager_connections:
  wifi:
    "Home WiFi":
      ssid: "MyNetwork"
      ifname: "wlan0"
      autoconnect: true
```

#### WireGuard Connections

```yaml
networkmanager_connections:
  wireguard:
    "WG Tunnel":
      ifname: "wg0"
      addresses: ["10.0.0.2/24"]
      dns: ["10.0.0.1"]
      private_key: "{{ vault_wg_private_key }}"
      peers:
        - public_key: "PEER_PUBLIC_KEY"
          endpoint: "vpn.example.com:51820"
          allowed_ips: "0.0.0.0/0"
          persistent_keepalive: 25
```

#### VPN Connections

```yaml
networkmanager_connections:
  vpn:
    "Company VPN":
      vpn:
        service-type: "org.freedesktop.NetworkManager.openvpn"
        gateway: "vpn.company.com"
      autoconnect: false
```

### Global DNS

Applied to connections without their own DNS config:

```yaml
networkmanager_global_dns4: []           # IPv4 DNS servers
networkmanager_global_dns4_search: []    # IPv4 search domains
networkmanager_global_dns4_options: []   # IPv4 options (edns0, trust-ad)
networkmanager_global_dns6: []           # IPv6 DNS servers
networkmanager_global_dns6_search: []
networkmanager_global_dns6_options: []
networkmanager_apply_global_dns: true    # enable global DNS application
```

### Dispatcher Features

```yaml
# Automatic timezone via ipapi.co
networkmanager_timezone_enabled: false

# VPN auto-connect on non-home networks
networkmanager_vpn_autoconnect:
  enabled: false
  connection: ''           # WireGuard connection name
  home_connections: []     # connections where VPN is not needed

# SMB shares (per-user auto-mount)
networkmanager_smb_shares: []
networkmanager_smb_credentials_dir: ".config/samba"

# SSHFS shares (auto-mount)
networkmanager_sshfs_shares: []
```

### Polkit

```yaml
networkmanager_polkit_enabled: false     # deploy polkit rule
networkmanager_polkit_group: 'wheel'     # group allowed to manage NM
```

### Unmanaged Devices

```yaml
networkmanager_unmanaged_devices:
  - mac: "00:22:68:1c:59:b1"
  - ifname: "docker0"
```

## Tags

| Tag                          | Scope                |
|------------------------------|----------------------|
| `networkmanager`             | All tasks            |
| `networkmanager:install`     | Package installation |
| `networkmanager:configure`   | Daemon configuration |
| `networkmanager:service`     | Service management   |
| `networkmanager:connections` | Connection profiles  |
| `networkmanager:polkit`      | Polkit rules         |
| `networkmanager:dispatcher`  | Dispatcher scripts   |

## Example Playbook

```yaml
- name: Configure NetworkManager
  hosts: workstations
  become: true
  roles:
    - role: marcstraube.common.networkmanager
      vars:
        networkmanager_dns: systemd-resolved
        networkmanager_wifi_powersave: disable
        networkmanager_log_level: WARN
        networkmanager_polkit_enabled: true

        networkmanager_connections:
          ethernet:
            "Office":
              ifname: "eth0"
              autoconnect: true
              toggle_wifi: true
              vpn: ["Company VPN"]
          wifi:
            "Home WiFi":
              ssid: "HomeNetwork"
              autoconnect: true
              dhcp_send_hostname_ipv4: false
          vpn:
            "Company VPN":
              vpn:
                service-type: org.freedesktop.NetworkManager.openvpn
                gateway: vpn.company.com
              autoconnect: false

        networkmanager_global_dns4: ["1.1.1.1", "8.8.8.8"]
        networkmanager_unmanaged_devices:
          - ifname: "docker0"

        networkmanager_timezone_enabled: true
```

## Testing

```bash
cd roles/networkmanager
molecule test
```

Driver: `vagrant` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

### Dispatcher Scripts

| Script                          | Trigger     | Purpose                                      |
|---------------------------------|-------------|----------------------------------------------|
| `09-timezone.sh`                | `up`        | Set timezone via IP geolocation              |
| `20-vpn-autoconnect.sh`         | `up`/`down` | Auto-activate WireGuard on non-home networks |
| `30-mount-smb.sh`               | `up`        | Mount SMB shares on connection up            |
| `40-umount-smb.sh`              | `down`      | Unmount SMB shares on connection down        |
| `30-mount-sshfs.sh`             | `up`        | Mount SSHFS shares                           |
| `40-umount-sshfs.sh`            | `down`      | Unmount SSHFS shares                         |
| `50-vpn-mount-smb.sh`           | `vpn-up`    | Mount SMB shares on VPN connect              |
| `60-vpn-umount-smb.sh`          | `vpn-down`  | Unmount SMB shares on VPN disconnect         |
| `99-wifi-auto-toggle.sh`        | `up`/`down` | Disable WiFi when ethernet connected         |
| `pre-down.d/30-umount-smb.sh`   | `pre-down`  | Graceful SMB unmount                         |
| `pre-down.d/30-umount-sshfs.sh` | `pre-down`  | Graceful SSHFS unmount                       |

## References

- [NetworkManager](https://networkmanager.dev/) — network configuration daemon
- [NetworkManager API Reference](https://networkmanager.dev/docs/api/latest/) — D-Bus API and configuration documentation

## License

MIT

## Author

Marc Straube
