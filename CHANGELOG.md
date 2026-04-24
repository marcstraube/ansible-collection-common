# Changelog

All notable changes to this collection will be documented in this file.

## [2.0.0](https://github.com/marcstraube/ansible-collection-common/compare/v1.2.0...v2.0.0) (2026-04-24)


### ⚠ BREAKING CHANGES

* Merge security and backup tasks into base_system.yml ([#96](https://github.com/marcstraube/ansible-collection-common/issues/96))
* Remove multiplexer variables from utils role ([#92](https://github.com/marcstraube/ansible-collection-common/issues/92))
* WireGuard role as standalone tunnel manager ([#83](https://github.com/marcstraube/ansible-collection-common/issues/83)) (#88)
* Change user_config_mode defaults to initial ([#62](https://github.com/marcstraube/ansible-collection-common/issues/62))
* Remove software-specific repos from package_management ([#61](https://github.com/marcstraube/ansible-collection-common/issues/61))
* Remove deprecated variable renames ([#60](https://github.com/marcstraube/ansible-collection-common/issues/60))

### Features

* Add base.yml playbook and standardize collection playbooks ([#50](https://github.com/marcstraube/ansible-collection-common/issues/50)) ([8253b35](https://github.com/marcstraube/ansible-collection-common/commit/8253b35193e495a3b3161701daec1b3f4a841cd3))
* Add multiplexer role (tmux, zellij) ([#52](https://github.com/marcstraube/ansible-collection-common/issues/52)) ([3a93988](https://github.com/marcstraube/ansible-collection-common/commit/3a9398887c6b04e2271736547b291f9ede73efd2))
* Add systemd-resolved role and reorganize networking stack ([#91](https://github.com/marcstraube/ansible-collection-common/issues/91)) ([ee8c08a](https://github.com/marcstraube/ansible-collection-common/commit/ee8c08a0ccdba368a9c2262db242e3f43cfce7b3))


### Bug Fixes

* Add --all argument to HSM sign script in apparmor ([#76](https://github.com/marcstraube/ansible-collection-common/issues/76)) ([efa068b](https://github.com/marcstraube/ansible-collection-common/commit/efa068bce9692a6a323e9d43698cc9a73ee8389b))
* Add missing fontconfig variable to fonts RedHat-9.yml ([#94](https://github.com/marcstraube/ansible-collection-common/issues/94)) ([42e612c](https://github.com/marcstraube/ansible-collection-common/commit/42e612c1c62437c8aac31624e49644ffd0b980e7))
* Auto-detect /tmp mount type in hardening role ([#87](https://github.com/marcstraube/ansible-collection-common/issues/87)) ([681d4ba](https://github.com/marcstraube/ansible-collection-common/commit/681d4ba603997cc6f8860dd939bdae7ba19366e0))
* BTRFS NoCOW tasks fail on missing directories ([#78](https://github.com/marcstraube/ansible-collection-common/issues/78)) ([33dcf5d](https://github.com/marcstraube/ansible-collection-common/commit/33dcf5d7225e18b13626c303700fb2a4920f1990))
* Correct README titles to collection FQCN format ([#51](https://github.com/marcstraube/ansible-collection-common/issues/51)) ([89a8885](https://github.com/marcstraube/ansible-collection-common/commit/89a8885d359b6648bd7971c21ef8e15385a07ca1))
* PHP role auto-version and Arch extension mapping ([#80](https://github.com/marcstraube/ansible-collection-common/issues/80)) ([cc0ef93](https://github.com/marcstraube/ansible-collection-common/commit/cc0ef930f1a3f8db3108c609ad2748fcb75f4c62))
* Remove _ansible_managed from JSON templates ([#72](https://github.com/marcstraube/ansible-collection-common/issues/72)) ([f89fb1a](https://github.com/marcstraube/ansible-collection-common/commit/f89fb1a136a3658add0a0fc08d7f50f45d7959ac))
* Reset FPM service name for system PHP in configure phase ([#85](https://github.com/marcstraube/ansible-collection-common/issues/85)) ([62e71c5](https://github.com/marcstraube/ansible-collection-common/commit/62e71c5475d5695d4ec10cb5c8045d95dc8f3573))
* Skip podman-docker when real Docker is installed ([#77](https://github.com/marcstraube/ansible-collection-common/issues/77)) ([c783a10](https://github.com/marcstraube/ansible-collection-common/commit/c783a10c24b8a51bc7a16efac6b64338307c2e76))
* Use official repo for python-podman on Arch ([#73](https://github.com/marcstraube/ansible-collection-common/issues/73)) ([5c881e7](https://github.com/marcstraube/ansible-collection-common/commit/5c881e7e50216a131452c8804208661a0a23299d))


### Code Refactoring

* Add playbook task files for infra delegation ([#71](https://github.com/marcstraube/ansible-collection-common/issues/71)) ([9a6e459](https://github.com/marcstraube/ansible-collection-common/commit/9a6e4590a23ee8215d554e502a52357d6390ba15))
* Change user_config_mode defaults to initial ([#62](https://github.com/marcstraube/ansible-collection-common/issues/62)) ([386e648](https://github.com/marcstraube/ansible-collection-common/commit/386e64848df75e31388f53363ffec0d07c571bb4))
* Merge security and backup tasks into base_system.yml ([#96](https://github.com/marcstraube/ansible-collection-common/issues/96)) ([f8770dc](https://github.com/marcstraube/ansible-collection-common/commit/f8770dc230a684e81abf15e00e0042def1bddd53))
* **php:** Move Sury repo setup from package_management to php role ([#45](https://github.com/marcstraube/ansible-collection-common/issues/45)) ([5616f89](https://github.com/marcstraube/ansible-collection-common/commit/5616f89f46e655005abc4f60cf185c30ddfa61c2))
* Remove deprecated variable renames ([#60](https://github.com/marcstraube/ansible-collection-common/issues/60)) ([61d7c3a](https://github.com/marcstraube/ansible-collection-common/commit/61d7c3a1e1af72dabb00ee0fdb3d588dbad79772))
* Remove multiplexer variables from utils role ([#92](https://github.com/marcstraube/ansible-collection-common/issues/92)) ([c9b0654](https://github.com/marcstraube/ansible-collection-common/commit/c9b06542aef07484dfd4ad7a3d524d3367120801))
* Remove software-specific repos from package_management ([#61](https://github.com/marcstraube/ansible-collection-common/issues/61)) ([e220163](https://github.com/marcstraube/ansible-collection-common/commit/e22016373df39dc6490e20ff60f8b4b782617f58))
* Rename base.yml to base_system.yml and remove networking.yml ([#56](https://github.com/marcstraube/ansible-collection-common/issues/56)) ([815e1c0](https://github.com/marcstraube/ansible-collection-common/commit/815e1c03d98576a0ee9d028c9bc43fadc453e1e1))
* Update backup.yml to default(true) ([#58](https://github.com/marcstraube/ansible-collection-common/issues/58)) ([be822df](https://github.com/marcstraube/ansible-collection-common/commit/be822dfcdec4b591ee8a94d970bc920d54d4a2fe))
* Update security.yml to default(true) and full role coverage ([#57](https://github.com/marcstraube/ansible-collection-common/issues/57)) ([25ca9dc](https://github.com/marcstraube/ansible-collection-common/commit/25ca9dc11f998ac25e3bbed13ce1755b2d85a872))
* WireGuard role as standalone tunnel manager ([#83](https://github.com/marcstraube/ansible-collection-common/issues/83)) ([#88](https://github.com/marcstraube/ansible-collection-common/issues/88)) ([b356db0](https://github.com/marcstraube/ansible-collection-common/commit/b356db0c05ff677b62667b07761f38744b061265))


### Documentation

* Add References sections to all role READMEs ([#63](https://github.com/marcstraube/ansible-collection-common/issues/63)) ([8c747ed](https://github.com/marcstraube/ansible-collection-common/commit/8c747edd5c63f7834e8077cfdec9559a111c4315))
* **podman:** Document userns modes and per-container override ([#41](https://github.com/marcstraube/ansible-collection-common/issues/41)) ([cdfc659](https://github.com/marcstraube/ansible-collection-common/commit/cdfc6590d46d368b457f385fd9ea4d0b281038e6))
* Update README role count and fix pre-commit performance ([#40](https://github.com/marcstraube/ansible-collection-common/issues/40)) ([eca6d8d](https://github.com/marcstraube/ansible-collection-common/commit/eca6d8d3b6bf4bfab675cbc4e266a8e7ddffa7d5))

## [1.2.0](https://github.com/marcstraube/ansible-collection-common/compare/v1.1.0...v1.2.0) (2026-04-19)


### Features

* Add network retry logic to all transient tasks ([#36](https://github.com/marcstraube/ansible-collection-common/issues/36)) ([4c23ddf](https://github.com/marcstraube/ansible-collection-common/commit/4c23ddfa91af40f22801572300addde97904f84d))
* **utils:** Add CLI file managers category ([#35](https://github.com/marcstraube/ansible-collection-common/issues/35)) ([69faeb4](https://github.com/marcstraube/ansible-collection-common/commit/69faeb42ce324bb856b1d191f0db85342b76b137))
* **utils:** Enable fd and fastfetch by default ([#37](https://github.com/marcstraube/ansible-collection-common/issues/37)) ([0656cec](https://github.com/marcstraube/ansible-collection-common/commit/0656cec447b934a57d6f013e2fa1c7bc41380d99))


### Bug Fixes

* Repair CI failures and add molecule gate job ([#33](https://github.com/marcstraube/ansible-collection-common/issues/33)) ([8152d97](https://github.com/marcstraube/ansible-collection-common/commit/8152d97c387fd50cca9580d465a277b780ad5f28))

## [1.1.0](https://github.com/marcstraube/ansible-collection-common/compare/v1.0.1...v1.1.0) (2026-04-17)


### Features

* Add Docker CE role ([#20](https://github.com/marcstraube/ansible-collection-common/issues/20)) ([#26](https://github.com/marcstraube/ansible-collection-common/issues/26)) ([ec443ea](https://github.com/marcstraube/ansible-collection-common/commit/ec443ea32f94a76d58cf7b6bdc5279bce12dcc82))
* Add PHP role with multi-version support, FPM, and Composer ([#15](https://github.com/marcstraube/ansible-collection-common/issues/15)) ([93323d0](https://github.com/marcstraube/ansible-collection-common/commit/93323d0e92f51b99e629586fe7e34371f21a490f))


### Bug Fixes

* Default firewalld_enabled to true for network-facing services ([5704fba](https://github.com/marcstraube/ansible-collection-common/commit/5704fba112d53a6d7c5d9652d2329767bc5337b4)), closes [#11](https://github.com/marcstraube/ansible-collection-common/issues/11)
* Remove unnecessary curl prerequisite from nodejs role ([281edc9](https://github.com/marcstraube/ansible-collection-common/commit/281edc988a0da4cee5c9d627fa857c53316fc5c0))
* Use explicit generic updater for galaxy.yml in release-please ([7887d36](https://github.com/marcstraube/ansible-collection-common/commit/7887d360d2750ebfa5cabeb1a283a4b12087ffd1))
* Use YAML-aware updater for galaxy.yml in release-please ([0aa7f80](https://github.com/marcstraube/ansible-collection-common/commit/0aa7f80c3eeeff825c811d3a564b67512a9c9ccb))


### Code Refactoring

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
