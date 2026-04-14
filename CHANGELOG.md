# Changelog

All notable changes to this collection will be documented in this file.

## [1.0.1](https://github.com/marcstraube/ansible-collection-common/compare/v1.0.0...v1.0.1) (2026-04-14)


### Bug Fixes

* Use generic updater for galaxy.yml version bump ([#6](https://github.com/marcstraube/ansible-collection-common/issues/6)) ([e829f20](https://github.com/marcstraube/ansible-collection-common/commit/e829f20a3c8362c55a83bab726039f02f1c0e724))
* Use markdownlint-cli2 ignores for CHANGELOG.md ([#5](https://github.com/marcstraube/ansible-collection-common/issues/5)) ([1cb1c54](https://github.com/marcstraube/ansible-collection-common/commit/1cb1c548a783ee80db656c7677b8fd89758f2818))


### Code Refactoring

* Standardize all 37 roles to unified coding conventions ([#2](https://github.com/marcstraube/ansible-collection-common/issues/2)) ([6d28947](https://github.com/marcstraube/ansible-collection-common/commit/6d28947b4874b0387bd2cc2ce31cc45cf076022c))

## [1.0.0] - 2026-04-10

### Summary

Initial public release of the `marcstraube.common` collection.

37 shared Ansible roles for multi-OS infrastructure management
supporting Arch Linux, Debian Trixie, and Rocky Linux 9/10.

### Roles

- **aide** - Advanced Intrusion Detection Environment
- **ansible** - Ansible, Molecule and linting tools
- **apparmor** - AppArmor mandatory access control
- **auditd** - Linux audit daemon
- **avahi** - mDNS/DNS-SD service discovery
- **base** - Base system (hostname, timezone, locale, kernel, systemd)
- **chrony** - NTP time synchronization
- **clamav** - ClamAV antivirus
- **editors** - Text editors (nano, vim, neovim) with per-user config
- **energy_management** - Power management (logind, PPD/TLP/tuned)
- **fail2ban** - Intrusion prevention
- **firejail** - Application sandboxing
- **firewalld** - Firewall management
- **fonts** - System and Nerd Fonts
- **gnupg** - GnuPG encryption and signing
- **graphics** - GPU drivers (Intel, NVIDIA, AMD)
- **hardening** - Kernel and filesystem hardening
- **hardware_tokens** - Nitrokey/YubiKey support
- **logrotate** - Log rotation
- **lynis** - Security auditing
- **networkmanager** - NetworkManager configuration
- **nodejs** - Node.js and NVM
- **openssh** - OpenSSH server and client
- **package_management** - Package managers (pacman, APT, DNF, AUR)
- **pam_hardening** - PAM security (pwquality, faillock)
- **pki** - PKI certificate management
- **podman** - Container runtime
- **python** - Python interpreter and tools
- **restic** - Backup with restic
- **rkhunter** - Rootkit detection
- **snmp** - SNMP monitoring agent
- **sudo** - Sudo configuration
- **sysctl** - Kernel parameter tuning
- **unbound** - DNS resolver
- **users** - User and group management
- **utils** - CLI utilities and monitoring tools
- **wireguard** - WireGuard VPN
