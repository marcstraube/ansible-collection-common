# marcstraube.common.libvirt

Install libvirt with QEMU/KVM so the host can run local virtual machines.

## Description

Installs the libvirt daemon, QEMU/KVM and dnsmasq from the distribution
packages, enables the libvirt daemons and starts the `default` NAT network.
Users listed in `libvirt_users` join the `libvirt` group and manage the
system instance (`qemu:///system`) without root, for example as a Vagrant
provider through `vagrant-libvirt`.

The role is opt-in: `libvirt_enabled` defaults to `false`.

Daemons per platform:

- Arch Linux, Debian Trixie: the monolithic `libvirtd`
- EL 9, EL 10: the modular `virtqemud`, `virtnetworkd`, `virtnodedevd`,
  `virtnwfilterd`, `virtsecretd` and `virtstoraged`

Each daemon is managed through its `.socket`, `-ro.socket` and `-admin.socket`
units and its `.service` unit.

## Requirements

- ansible-core >= 2.19
- x86_64 host; hardware virtualization (Intel VT-x or AMD-V) enabled in the
  firmware for KVM acceleration

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

| Variable                  | Default | Description                          |
|---------------------------|---------|--------------------------------------|
| `libvirt_enabled`         | `false` | Enable the libvirt role              |
| `libvirt_service_enabled` | `true`  | Enable and start the libvirt service |

`libvirt_service_enabled: false` stops and disables the daemon sockets and
services; the default network is then left alone.

### Users

| Variable        | Default | Description                         |
|-----------------|---------|-------------------------------------|
| `libvirt_users` | `[]`    | Users to add to the `libvirt` group |

```yaml
libvirt_users:
  - 'johndoe'
```

Members of the `libvirt` group manage system VMs without root through the
polkit rule libvirt ships. A new membership takes effect at the next login.

### Default Network

| Variable                            | Default | Description                                          |
|-------------------------------------|---------|------------------------------------------------------|
| `libvirt_default_network_autostart` | `true`  | Start the `default` NAT network and enable autostart |

With `true`, the role defines the `default` network from the definition the
packages ship when it is missing, enables its autostart and starts it. With
`false`, it disables the autostart and stops the network; the definition is
kept.

## Tags

| Tag                 | Scope                                 |
|---------------------|---------------------------------------|
| `libvirt`           | All tasks                             |
| `libvirt:install`   | Package installation                  |
| `libvirt:configure` | `libvirt` group and user membership   |
| `libvirt:service`   | Daemon sockets and services           |
| `libvirt:network`   | Default NAT network                   |

## Example Playbook

```yaml
- name: Include libvirt role
  ansible.builtin.include_role:
    name: marcstraube.common.libvirt
  vars:
    libvirt_enabled: true
    libvirt_users:
      - 'johndoe'
  tags:
    - libvirt
```

## Testing

```bash
cd roles/libvirt
molecule test
```

Driver: `vagrant` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

The test VMs have no nested virtualization; the scenario covers the packages,
the group membership, the daemon units and the default network, not running
guests.

## Notes

### Firewall

libvirt creates the firewall rules of its own networks (NAT, DHCP and DNS for
guests) when a network starts; the role adds no rules. On Arch Linux the role
installs `nftables`, which libvirt prefers as firewall backend. On Debian and
EL the network driver package pulls in the backend. With firewalld, libvirt
places its bridges in the `libvirt` zone.

### Default Network Address

The `default` network uses `192.168.122.0/24`. When that subnet is already in
use on the host, starting the network fails; change the network with
`virsh net-edit default` or set `libvirt_default_network_autostart: false`.

## References

- [libvirt](https://libvirt.org/) — virtualization API and daemons
- [libvirt daemons](https://libvirt.org/daemons.html) — monolithic and modular daemon modes
- [libvirt networking](https://wiki.libvirt.org/VirtualNetworking.html) — virtual networks and NAT
- [vagrant-libvirt](https://vagrant-libvirt.github.io/vagrant-libvirt/) — Vagrant provider for libvirt

## License

MIT

## Author

Marc Straube
