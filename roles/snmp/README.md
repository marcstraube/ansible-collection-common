# marcstraube.common.snmp

Install and configure the Net-SNMP agent (snmpd) for network monitoring.

## Description

Manages Net-SNMP agent installation and configuration with SNMPv2c/v3 support,
VACM access control, disk/process/load monitoring, AgentX sub-agent support,
firewalld integration, and AppArmor profile enforcement.

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

See [`defaults/main.yml`](defaults/main.yml) for all variables with descriptions.

### Key Variables

| Variable                 | Default               | Description                           |
|--------------------------|-----------------------|---------------------------------------|
| `snmp_enabled`           | `true`                | Enable the snmp role                  |
| `snmp_service_enabled`   | `true`                | Enable and start the snmp service     |
| `snmp_listen_addresses`  | `['127.0.0.1']`       | Addresses snmpd listens on (UDP 161)  |
| `snmp_sys_contact`       | `'admin@example.com'` | sysContact OID value                  |
| `snmp_sys_location`      | `'Server Room'`       | sysLocation OID value                 |
| `snmp_v2c_enabled`       | `false`               | Enable SNMPv2c community-based access |
| `snmp_v3_enabled`        | `true`                | Enable SNMPv3 user-based security     |
| `snmp_v3_users`          | `[]`                  | SNMPv3 user definitions (auth/priv)   |
| `snmp_firewalld_enabled` | `false`               | Enable firewalld rule for SNMP        |
| `snmp_firewalld_zone`    | `'public'`            | Firewalld zone for SNMP service       |
| `snmp_apparmor_enabled`  | `false`               | Enable AppArmor profile enforcement   |
| `snmp_agentx_enabled`    | `false`               | Enable AgentX sub-agent support       |

### SNMPv3 Users

```yaml
snmp_v3_users:
  - username: 'monitoring'
    auth_protocol: 'SHA'          # MD5, SHA, SHA-224, SHA-256, SHA-384, SHA-512
    auth_passphrase: '{{ vault_snmp_v3_auth_passphrase }}'
    priv_protocol: 'AES'          # DES, AES, AES-192, AES-256
    priv_passphrase: '{{ vault_snmp_v3_priv_passphrase }}'
    access: 'rouser'              # rouser or rwuser
    security_level: 'authPriv'    # noAuthNoPriv, authNoPriv, authPriv
```

Passphrases must be at least 8 characters. SNMPv3 users are managed via a
checksum-based idempotency mechanism — the persistent store is only wiped
when the user list changes.

### VACM Access Control

For fine-grained access control using the traditional com2sec/group/access model:

```yaml
snmp_com2sec:
  - sec_name: 'mynet'
    source: '10.0.0.0/24'
    community: '{{ vault_snmp_community }}'

snmp_groups:
  - name: 'mygroup'
    version: 'v2c'
    sec_name: 'mynet'

snmp_access:
  - group: 'mygroup'
    context: ''
    match: 'any'
    level: 'noauth'
    prefix: 'exact'
    read: 'all'
    write: 'none'
    notify: 'none'
```

### Monitoring

```yaml
snmp_disk_monitors:
  - path: '/'
    min_free: '10%'

snmp_process_monitors:
  - name: 'sshd'
    min: 1

snmp_load_thresholds:
  load1: 12
  load5: 10
  load15: 8
```

## Tags

| Tag              | Scope                                    |
|------------------|------------------------------------------|
| `snmp`           | All tasks                                |
| `snmp:install`   | Package installation                     |
| `snmp:configure` | Configuration and SNMPv3 user management |
| `snmp:service`   | Service enable/disable                   |
| `snmp:firewall`  | Firewalld rules                          |
| `snmp:apparmor`  | AppArmor profile enforcement             |

## Example Playbook

```yaml
- name: Configure SNMP monitoring
  hosts: servers
  become: true

  vars:
    snmp_enabled: true
    snmp_listen_addresses:
      - '0.0.0.0'
    snmp_v3_users:
      - username: 'monitoring'
        auth_protocol: 'SHA-256'
        auth_passphrase: '{{ vault_snmp_v3_auth_passphrase }}'
        priv_protocol: 'AES'
        priv_passphrase: '{{ vault_snmp_v3_priv_passphrase }}'
    snmp_firewalld_enabled: true

  roles:
    - marcstraube.common.snmp
```

## Testing

```bash
cd roles/snmp
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

### Arch Linux

The upstream `snmpd.service` unit hardcodes `udp: udp6:` as listen addresses
in `ExecStart`. This role deploys a systemd override to remove that, allowing
`agentaddress` directives in `snmpd.conf` to control listen addresses.

### AppArmor

AppArmor enforcement is available on Arch Linux and Debian (not RedHat/Rocky
which uses SELinux). Enable with `snmp_apparmor_enabled: true`.

## License

MIT

## Author

Marc Straube
