# Ansible Collection: marcstraube.common

[![CI](https://github.com/marcstraube/ansible-collection-common/workflows/CI/badge.svg)](https://github.com/marcstraube/ansible-collection-common/actions)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Ko-fi](https://img.shields.io/badge/Ko--fi-Support-ff5e5b?logo=ko-fi&logoColor=white)](https://ko-fi.com/marcstraube)

## Description

Shared Ansible roles for multi-OS infrastructure management.
Roles covering base system, security, networking, package management,
development toolchains, user management, editors, and more.

## Supported Platforms

- Arch Linux (rolling)
- Debian Trixie (13)
- EL 9 (Rocky, Alma, RHEL)
- EL 10 (Rocky, Alma, RHEL)

## Requirements

- ansible-core >= 2.19

## Included Roles

### Base System

| Role                  | Description                                               |
| --------------------- | --------------------------------------------------------- |
| **base**              | Hostname, timezone, locale, kernel, systemd               |
| **users**             | User/group management with SSH keys                       |
| **sudo**              | Sudo configuration                                        |
| **editors**           | Text editors (nano, vim, neovim) with per-user config     |
| **multiplexer**       | Terminal multiplexers (tmux, zellij) with per-user config |
| **shell**             | Shells (zsh, bash, fish), Starship prompt, modern CLI     |
| **utils**             | CLI utilities and monitoring tools                        |
| **fonts**             | System and Nerd Fonts                                     |
| **energy_management** | Power management (logind, PPD/TLP/tuned)                  |
| **graphics**          | GPU drivers (Intel, NVIDIA, AMD)                          |
| **storage**           | Data disks (LUKS2, btrfs/snapper, LVM, ext4), fs tools    |

### Package Management

| Role                   | Description                                    |
| ---------------------- | ---------------------------------------------- |
| **package_management** | pacman, APT, DNF, AUR helper (paru), reflector |

### Development

| Role                  | Description                                         |
| --------------------- | --------------------------------------------------- |
| **git**               | Git, git-lfs, GitHub/GitLab CLIs, per-user config   |
| **python**            | Python interpreter, pipx and tooling                |
| **nodejs**            | Node.js and NVM                                     |
| **php**               | PHP interpreter and extensions                      |
| **rust**              | Rust toolchain (packages or rustup)                 |
| **go**                | Go toolchain                                        |
| **java**              | OpenJDK                                             |
| **build_tools**       | make, CMake, Meson, Ninja, C/C++ toolchain          |
| **security_scanners** | Socket CLI, trivy, gitleaks, osv-scanner, syft      |
| **ansible_tools**     | Ansible, Molecule and linting tools                 |
| **kubernetes**        | Kubernetes client tooling (kubectl, helm, minikube) |

### Networking

| Role               | Description                   |
| ------------------ | ----------------------------- |
| **resolved**       | systemd-resolved DNS resolver |
| **networkmanager** | NetworkManager configuration  |
| **firewalld**      | Firewall management           |
| **wireguard**      | WireGuard VPN                 |
| **unbound**        | DNS resolver with DNSSEC      |
| **avahi**          | mDNS/DNS-SD service discovery |
| **openssh**        | OpenSSH server and client     |

### Security

| Role              | Description                              |
| ----------------- | ---------------------------------------- |
| **hardening**     | Kernel and filesystem hardening          |
| **sysctl**        | Kernel parameter tuning                  |
| **pam_hardening** | PAM security (pwquality, faillock)       |
| **apparmor**      | AppArmor mandatory access control        |
| **firejail**      | Application sandboxing                   |
| **fail2ban**      | Intrusion prevention                     |
| **auditd**        | Linux audit daemon                       |
| **aide**          | Advanced Intrusion Detection Environment |
| **lynis**         | Security auditing                        |
| **rkhunter**      | Rootkit detection                        |
| **clamav**        | ClamAV antivirus                         |
| **wazuh_agent**   | Wazuh agent with manager enrollment      |

### Crypto & Authentication

| Role                | Description                  |
| ------------------- | ---------------------------- |
| **gnupg**           | GnuPG encryption and signing |
| **pki**             | PKI certificate management   |
| **hardware_tokens** | Nitrokey/YubiKey support     |

### Services & Infrastructure

| Role          | Description              |
| ------------- | ------------------------ |
| **chrony**    | NTP time synchronization |
| **logrotate** | Log rotation             |
| **mta**       | Send-only mail (Postfix) |
| **docker**    | Docker container runtime |
| **podman**    | Podman container runtime |
| **libvirt**   | libvirt with QEMU/KVM    |
| **snmp**      | SNMP monitoring agent    |
| **restic**    | Backup with restic       |

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy collection install marcstraube.common
```

### From Git

```bash
ansible-galaxy collection install git+https://github.com/marcstraube/ansible-collection-common.git,main
```

### Requirements File

```yaml
# requirements.yml
collections:
  - name: marcstraube.common
    version: ">=3.0.0"
```

## Usage

All roles use `include_role` with boolean toggles:

```yaml
- name: Configure system
  hosts: all
  become: true

  tasks:
    - name: Include base role
      ansible.builtin.include_role:
        name: marcstraube.common.base
      when: base_enabled | default(true) | bool

    - name: Include openssh role
      ansible.builtin.include_role:
        name: marcstraube.common.openssh
      when: openssh_enabled | default(true) | bool
```

`playbooks/tasks/base_system.yml` imports every role of the collection with
`import_role` and the same toggles, so a play can include that task file
instead of listing the roles itself.

Each role documents its variables in `defaults/main.yml` and `roles/<role>/README.md`.

## Testing

Roles are tested with Molecule using Podman containers across all supported platforms.

```bash
cd roles/<role>
molecule test
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for development setup and guidelines.
Pull requests target the `develop` branch; `main` holds the released state.

## Support

If you find this collection useful, consider [supporting its development](https://github.com/sponsors/marcstraube).

## License

MIT

## Author

Marc Straube (<email@marcstraube.de>)
