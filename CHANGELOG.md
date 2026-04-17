# Changelog

All notable changes to this collection will be documented in this file.

## [1.1.0](https://github.com/marcstraube/ansible-collection-common/compare/v1.0.1...v1.1.0) (2026-04-17)


### Features

* Add Docker CE role ([#20](https://github.com/marcstraube/ansible-collection-common/issues/20)) ([#26](https://github.com/marcstraube/ansible-collection-common/issues/26)) ([ec443ea](https://github.com/marcstraube/ansible-collection-common/commit/ec443ea32f94a76d58cf7b6bdc5279bce12dcc82))
* Add PHP role with multi-version support, FPM, and Composer ([5ff71a1](https://github.com/marcstraube/ansible-collection-common/commit/5ff71a1b04d2ce30ec1061c0e05d5a9d916924ff))
* Add PHP role with multi-version support, FPM, and Composer ([#15](https://github.com/marcstraube/ansible-collection-common/issues/15)) ([93323d0](https://github.com/marcstraube/ansible-collection-common/commit/93323d0e92f51b99e629586fe7e34371f21a490f))


### Bug Fixes

* Default firewalld_enabled to true for network-facing services ([26eda46](https://github.com/marcstraube/ansible-collection-common/commit/26eda46ded3c6d3d67152b737495f31ad5a76c65))
* Default firewalld_enabled to true for network-facing services ([5704fba](https://github.com/marcstraube/ansible-collection-common/commit/5704fba112d53a6d7c5d9652d2329767bc5337b4)), closes [#11](https://github.com/marcstraube/ansible-collection-common/issues/11)
* Remove unnecessary curl prerequisite from nodejs role ([e6da9fd](https://github.com/marcstraube/ansible-collection-common/commit/e6da9fded0a548a9763b3313187ba191b8cbbec6))
* Remove unnecessary curl prerequisite from nodejs role ([281edc9](https://github.com/marcstraube/ansible-collection-common/commit/281edc988a0da4cee5c9d627fa857c53316fc5c0))
* Use explicit generic updater for galaxy.yml in release-please ([8437186](https://github.com/marcstraube/ansible-collection-common/commit/8437186c97ec30ebdf095636a64203699395db62))
* Use explicit generic updater for galaxy.yml in release-please ([7887d36](https://github.com/marcstraube/ansible-collection-common/commit/7887d360d2750ebfa5cabeb1a283a4b12087ffd1))
* Use YAML-aware updater for galaxy.yml in release-please ([aa7f030](https://github.com/marcstraube/ansible-collection-common/commit/aa7f03017d3a9c5895a8732268c9e06e53451e23))
* Use YAML-aware updater for galaxy.yml in release-please ([0aa7f80](https://github.com/marcstraube/ansible-collection-common/commit/0aa7f80c3eeeff825c811d3a564b67512a9c9ccb))


### Code Refactoring

* Unify molecule prepare.yml and add dependency conventions ([a559e6c](https://github.com/marcstraube/ansible-collection-common/commit/a559e6c5f88d4d0e57a9a19cf6777c07d8d3cdb1))
* Unify molecule prepare.yml and add dependency conventions ([5cfd469](https://github.com/marcstraube/ansible-collection-common/commit/5cfd4699bc7722800ee2441de40ea9b77447fc94)), closes [#17](https://github.com/marcstraube/ansible-collection-common/issues/17) [#21](https://github.com/marcstraube/ansible-collection-common/issues/21)

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
