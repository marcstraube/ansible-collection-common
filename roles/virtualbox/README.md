# Ansible Role: virtualbox

Install Oracle VirtualBox, provide its kernel modules and grant users
access, so the host can run local virtual machines, for example as a
Vagrant provider.

On Arch Linux the role installs VirtualBox from the official repository and
builds the modules with DKMS for every installed kernel. On Debian and EL it
installs the selected branch from the Oracle repository and lets the
`vboxdrv` service build and load the modules.

## Requirements

- ansible-core >= 2.19
- Debian and EL: the Oracle VirtualBox repository, configured through
  `marcstraube.common.package_management` (see
  [Oracle Repository](#oracle-repository))
- EL: EPEL (`dnf_epel_enabled` in `marcstraube.common.package_management`),
  which provides libraries the Oracle packages need
- A CPU with hardware virtualization (Intel VT-x or AMD-V) enabled in the
  firmware to run virtual machines

### Required Collections

```yaml
collections:
  - community.general
```

## Supported Platforms

| Platform                  | Notes                                     |
| ------------------------- | ----------------------------------------- |
| Arch Linux                | `virtualbox` with `virtualbox-host-dkms`  |
| Debian Trixie             | Oracle repository, `virtualbox-<version>` |
| EL 9 (Rocky, Alma, RHEL)  | Oracle repository, `VirtualBox-<version>` |
| EL 10 (Rocky, Alma, RHEL) | Oracle repository, `VirtualBox-<version>` |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu,
Mint, Fedora) should work but are not actively tested. Use
distro-specific vars overrides if needed.

## Role Variables

### Role Control

| Variable                     | Default | Description                |
| ---------------------------- | ------- | -------------------------- |
| `virtualbox_enabled`         | `false` | Enable the virtualbox role |
| `virtualbox_service_enabled` | `true`  | Enable and start `vboxdrv` |

`virtualbox_service_enabled` manages the `vboxdrv` service on Debian and EL.
Arch Linux has no such service: `virtualbox-host-dkms` loads the modules at
boot through `modules-load.d`, and the role loads them for the running
kernel.

### Installation

| Variable             | Default | Description                        |
| -------------------- | ------- | ---------------------------------- |
| `virtualbox_version` | `'7.2'` | VirtualBox branch on Debian and EL |

The branch selects the Oracle package (`virtualbox-7.2` on Debian,
`VirtualBox-7.2` on EL). Updates within the branch arrive with the regular
system upgrade. A changed branch removes the installed one and installs the
selected one; virtual machines and their settings stay in place. Arch Linux
ignores the variable and installs the current release from the official
repository.

### Users

| Variable           | Default | Description                          |
| ------------------ | ------- | ------------------------------------ |
| `virtualbox_users` | `[]`    | Users added to the `vboxusers` group |

Virtual machines run without the group membership; `vboxusers` grants
access to USB devices for passthrough. The accounts must exist, for example
created by `marcstraube.common.users`. A new membership applies from the
next login.

## Tags

| Tag                    | Description                                     |
| ---------------------- | ----------------------------------------------- |
| `virtualbox`           | All virtualbox tasks                            |
| `virtualbox:install`   | Kernel headers, build dependencies and packages |
| `virtualbox:configure` | `vboxusers` group membership                    |
| `virtualbox:service`   | `vboxdrv` service and kernel modules            |

## Example Playbook

```yaml
- hosts: workstations
  become: true
  vars:
    apt_custom_repos:
      - name: 'virtualbox'
        uris: 'https://download.virtualbox.org/virtualbox/debian'
        suites: 'trixie'
        components: 'contrib'
        gpg_key: 'https://www.virtualbox.org/download/oracle_vbox_2016.asc'
        architectures: 'amd64'

    dnf_custom_repos:
      - name: 'virtualbox'
        description: 'Oracle VirtualBox'
        baseurl: 'https://download.virtualbox.org/virtualbox/rpm/el/$releasever/$basearch'
        gpgkey: 'https://www.virtualbox.org/download/oracle_vbox_2016.asc'
        gpgcheck: true

    virtualbox_enabled: true
    virtualbox_users:
      - 'alice'
  roles:
    - marcstraube.common.package_management
    - marcstraube.common.virtualbox
```

## Testing

```bash
cd roles/virtualbox
MOLECULE_PLATFORM_NAME=virtualbox-archlinux molecule test -p virtualbox-archlinux
```

Driver: `vagrant` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

The kernel modules need a VM. `prepare.yml` upgrades the system, boots the
upgraded kernel and adds the Oracle repository on Debian and EL. Verify
checks the packages, the kernel headers, the installed branch, the
`vboxusers` membership, the loaded `vboxdrv` module and, on Debian and EL,
the `vboxdrv` service.

## Notes

### Oracle Repository

The role does not configure package repositories. On Debian and EL, add the
Oracle repository through the `marcstraube.common.package_management`
inventory:

```yaml
# Debian (deb822)
apt_custom_repos:
  - name: 'virtualbox'
    uris: 'https://download.virtualbox.org/virtualbox/debian'
    suites: 'trixie'
    components: 'contrib'
    gpg_key: 'https://www.virtualbox.org/download/oracle_vbox_2016.asc'
    architectures: 'amd64'

# EL 9 and EL 10
dnf_custom_repos:
  - name: 'virtualbox'
    description: 'Oracle VirtualBox'
    baseurl: 'https://download.virtualbox.org/virtualbox/rpm/el/$releasever/$basearch'
    gpgkey: 'https://www.virtualbox.org/download/oracle_vbox_2016.asc'
    gpgcheck: true
```

Oracle publishes Debian packages for `amd64` only. `$releasever` resolves
to the major version (`9`, `10`), which matches the directories of the
Oracle repository.

### Kernel Modules

- Arch Linux: the role installs the headers of every installed kernel of
  the official repository (`linux`, `linux-lts`, `linux-zen`,
  `linux-hardened`, `linux-rt`, `linux-rt-lts`), so DKMS builds the
  modules for each of them, also after kernel updates. After an upgrade
  that replaced the running kernel, the modules load after the next reboot.
- Debian: the role installs the headers of the running kernel and the
  headers metapackage of its flavor (`linux-headers-amd64`,
  `linux-headers-cloud-amd64`), which follows kernel updates.
- EL: the role installs `kernel-devel` of the running kernel; dnf installs
  the matching `kernel-devel` with every kernel update.
- Debian and EL: the `vboxdrv` service builds the modules for the running
  kernel when they are missing and loads them. Run the role after a reboot
  into the current kernel: the repositories carry only the headers of
  current kernels.

### Secure Boot

Kernels that enforce module signatures, as Debian and EL kernels do under
Secure Boot, refuse unsigned modules, and `vboxdrv` fails to load. The role
does not manage signing keys. Create a Machine Owner Key, enroll it with
`mokutil --import` and let the module build sign with it:

- Arch Linux: the kernels load unsigned modules under Secure Boot as well;
  they enforce signatures only with the kernel parameter
  `module.sig_enforce=1` or `lockdown=integrity`. DKMS then signs the
  modules with the key and certificate set in `mok_signing_key` and
  `mok_certificate` in `/etc/dkms/framework.conf`. The kernel verifies
  modules against Machine Owner Keys only, not against keys in the UEFI
  `db`, and only after `mokutil --trust-mok`; this requires booting through
  shim.
- Debian and EL: the `vboxdrv` setup signs the modules it builds; see the
  VirtualBox manual, section *Kernel Modules and UEFI Secure Boot*, for the
  key locations it uses.

### Firewall

VirtualBox networking (NAT, host-only, bridged) needs no open ports on the
host. Ports forwarded into a virtual machine through NAT listen on the host
and need a rule in the `marcstraube.common.firewalld` inventory when other
machines connect to them.

## References

- [VirtualBox Linux downloads and repositories](https://www.virtualbox.org/wiki/Linux_Downloads)
- [VirtualBox manual: installing on Linux hosts](https://www.virtualbox.org/manual/topics/installation.html)
- [Arch Wiki: VirtualBox](https://wiki.archlinux.org/title/VirtualBox)
- [VirtualBox changelog](https://www.virtualbox.org/wiki/Changelog)

## License

MIT

## Author

Marc Straube
