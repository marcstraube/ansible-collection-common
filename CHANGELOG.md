# Changelog

All notable changes to this collection will be documented in this file.

## [2.2.1](https://github.com/marcstraube/ansible-collection-common/compare/v2.2.0...v2.2.1) (2026-07-10)


### Bug Fixes

* replace hardcoded aur_builder with aur_builder_user variable ([#265](https://github.com/marcstraube/ansible-collection-common/issues/265)) ([bc91bad](https://github.com/marcstraube/ansible-collection-common/commit/bc91bad07411e7699dec01d3dd5f035b411c01ca))

## [2.2.0](https://github.com/marcstraube/ansible-collection-common/compare/v2.1.1...v2.2.0) (2026-07-07)


### Features

* add wait_for_connection recovery timeout to firewalld and wireguard ([#258](https://github.com/marcstraube/ansible-collection-common/issues/258)) ([3c6685b](https://github.com/marcstraube/ansible-collection-common/commit/3c6685bf508ce6f80775e5b6aa81b2c8b3e21299))
* **networkmanager:** add zone option for firewalld zone assignment ([#234](https://github.com/marcstraube/ansible-collection-common/issues/234)) ([98d7b03](https://github.com/marcstraube/ansible-collection-common/commit/98d7b030d433895e96cdc04f9f69a2774bdcc3ba))
* **openssh:** add optional SSH client tools (sshm, mosh, sshfs, autossh) ([#261](https://github.com/marcstraube/ansible-collection-common/issues/261)) ([854ed63](https://github.com/marcstraube/ansible-collection-common/commit/854ed63858c72d8227c571976bf3ad557fd31277))
* **package_management:** add makepkg_devtools_enabled toggle ([#243](https://github.com/marcstraube/ansible-collection-common/issues/243)) ([8db4180](https://github.com/marcstraube/ansible-collection-common/commit/8db41802d0e73fb90111e0cfb695579a6fea27d0))
* **package_management:** add makepkg_nvchecker_enabled toggle ([#250](https://github.com/marcstraube/ansible-collection-common/issues/250)) ([96272c4](https://github.com/marcstraube/ansible-collection-common/commit/96272c48799ce0a0b3420424a3f5c06a9afc7ab4)), closes [#231](https://github.com/marcstraube/ansible-collection-common/issues/231)
* **php:** add pcov to the extension map ([#253](https://github.com/marcstraube/ansible-collection-common/issues/253)) ([9410da8](https://github.com/marcstraube/ansible-collection-common/commit/9410da860f01a05bc87d414ec27f110440c5586f)), closes [#217](https://github.com/marcstraube/ansible-collection-common/issues/217)


### Bug Fixes

* **networkmanager:** reload daemon config instead of restarting to preserve SSH ([#239](https://github.com/marcstraube/ansible-collection-common/issues/239)) ([c2f5bdc](https://github.com/marcstraube/ansible-collection-common/commit/c2f5bdc2a5f98d6b1218d7282b2e63d2ef3d680e)), closes [#238](https://github.com/marcstraube/ansible-collection-common/issues/238)
* **nodejs:** don't fail verify in check mode when Node.js is absent ([#237](https://github.com/marcstraube/ansible-collection-common/issues/237)) ([5563c39](https://github.com/marcstraube/ansible-collection-common/commit/5563c397a1c819505096a64e07ee3fe93cd540ab))
* **package_management:** grant aur_shared access to paru clone dir via ACLs ([#262](https://github.com/marcstraube/ansible-collection-common/issues/262)) ([7ffb05e](https://github.com/marcstraube/ansible-collection-common/commit/7ffb05e9fd658e5b4e77bbf085da03e093bc51ef))
* **php:** recompute per-version Remi facts in configure and service phases ([#255](https://github.com/marcstraube/ansible-collection-common/issues/255)) ([8730123](https://github.com/marcstraube/ansible-collection-common/commit/8730123a9afb523f43c7623aec61c0109a557cca)), closes [#254](https://github.com/marcstraube/ansible-collection-common/issues/254) [#192](https://github.com/marcstraube/ansible-collection-common/issues/192)
* **php:** resolve Remi extensions to versioned SCL package names ([#252](https://github.com/marcstraube/ansible-collection-common/issues/252)) ([a54b7b3](https://github.com/marcstraube/ansible-collection-common/commit/a54b7b3049caa08dbb58172bf36baa79eddf8a92)), closes [#246](https://github.com/marcstraube/ansible-collection-common/issues/246)


### Documentation

* **package_management:** fix Debian repo table markdownlint line-length ([#242](https://github.com/marcstraube/ansible-collection-common/issues/242)) ([0fb6754](https://github.com/marcstraube/ansible-collection-common/commit/0fb6754d8899e42f40883a94520b7c8cd8e7314b))

## [2.1.1](https://github.com/marcstraube/ansible-collection-common/compare/v2.1.0...v2.1.1) (2026-06-12)


### Bug Fixes

* **base:** use OS-specific binary and path for GRUB regeneration handler ([#219](https://github.com/marcstraube/ansible-collection-common/issues/219)) ([84007fc](https://github.com/marcstraube/ansible-collection-common/commit/84007fc57147575dd6601ab4a831d812a58147c2)), closes [#218](https://github.com/marcstraube/ansible-collection-common/issues/218)
* **hardening:** restart PrivateTmp services after /var/tmp bind change ([#224](https://github.com/marcstraube/ansible-collection-common/issues/224)) ([8384b84](https://github.com/marcstraube/ansible-collection-common/commit/8384b842e9d4e9e3adbfd2dc7d9f9f852cb3a558)), closes [#223](https://github.com/marcstraube/ansible-collection-common/issues/223)
* **nodejs:** pre-create global node_modules directory with mode 0755 ([#226](https://github.com/marcstraube/ansible-collection-common/issues/226)) ([41965d3](https://github.com/marcstraube/ansible-collection-common/commit/41965d35482830310a022b2fba4e4a5cbf36c58e)), closes [#225](https://github.com/marcstraube/ansible-collection-common/issues/225)
* **python:** honor optional HTTP(S) proxy for pipx bootstrap pip install ([#222](https://github.com/marcstraube/ansible-collection-common/issues/222)) ([90ddaa2](https://github.com/marcstraube/ansible-collection-common/commit/90ddaa20b58767001bbd2afd19b29199f322d081)), closes [#221](https://github.com/marcstraube/ansible-collection-common/issues/221)

## [2.1.0](https://github.com/marcstraube/ansible-collection-common/compare/v2.0.0...v2.1.0) (2026-06-11)


### Features

* **docker:** make Wait-for-connection timeout configurable ([#202](https://github.com/marcstraube/ansible-collection-common/issues/202)) ([0ba3e2b](https://github.com/marcstraube/ansible-collection-common/commit/0ba3e2bece8e38edb48387b27c9868e1f89702db)), closes [#165](https://github.com/marcstraube/ansible-collection-common/issues/165)
* **package_management:** apt_non_free toggle for Debian default sources ([#195](https://github.com/marcstraube/ansible-collection-common/issues/195)) ([6b58470](https://github.com/marcstraube/ansible-collection-common/commit/6b5847097355e8da4309ea826188324fe107bd08))
* **roles:** add minor-version level to with_first_found vars lookup ([#206](https://github.com/marcstraube/ansible-collection-common/issues/206)) ([0ab916e](https://github.com/marcstraube/ansible-collection-common/commit/0ab916eac387fe6f97eb408da8f54a30a7be8ba8)), closes [#135](https://github.com/marcstraube/ansible-collection-common/issues/135)
* **utils:** add Archive / Compression section ([#196](https://github.com/marcstraube/ansible-collection-common/issues/196)) ([cbfc7aa](https://github.com/marcstraube/ansible-collection-common/commit/cbfc7aa9833a7ede9dff193bceef2f4e098b2a8c))


### Bug Fixes

* **aide:** drop stale vars/RedHat-9.yml after EPEL AIDE 0.19 upgrade ([#208](https://github.com/marcstraube/ansible-collection-common/issues/208)) ([6c32fa3](https://github.com/marcstraube/ansible-collection-common/commit/6c32fa39f8e3ada42e9e07c317b1755338829067)), closes [#207](https://github.com/marcstraube/ansible-collection-common/issues/207)
* **base:** default base_bootloader per OS (grub on RHEL/Debian, systemd-boot on Arch) ([#180](https://github.com/marcstraube/ansible-collection-common/issues/180)) ([c9d79d9](https://github.com/marcstraube/ansible-collection-common/commit/c9d79d9c4a7d44063140aab765827169ed4c834e))
* **docker:** refresh dnf cache after repo add and skip install when repo missing ([#189](https://github.com/marcstraube/ansible-collection-common/issues/189)) ([92d6b91](https://github.com/marcstraube/ansible-collection-common/commit/92d6b919e08bdff38744108b8e2d734f83c95b34))
* **gnupg:** make paperkey a Layer-1 no-op on EL (not in EPEL 9/10) ([#187](https://github.com/marcstraube/ansible-collection-common/issues/187)) ([6e4fb4c](https://github.com/marcstraube/ansible-collection-common/commit/6e4fb4cefa3c5fcb67ba32239ed27d8f76a75291))
* **hardening:** preserve existing /tmp instead of force-converting to tmpfs ([#183](https://github.com/marcstraube/ansible-collection-common/issues/183)) ([d48f173](https://github.com/marcstraube/ansible-collection-common/commit/d48f173a8875cba7d2073fdd33d0442f73f4eb69))
* **nodejs:** probe node/npm versions in check mode ([#199](https://github.com/marcstraube/ansible-collection-common/issues/199)) ([6829a4c](https://github.com/marcstraube/ansible-collection-common/commit/6829a4c8ddc9965efcd22ff33727fd5e680114b7))
* **php:** split Arch extension map by official vs AUR layout ([#201](https://github.com/marcstraube/ansible-collection-common/issues/201)) ([c5a9c87](https://github.com/marcstraube/ansible-collection-common/commit/c5a9c872d491123be37b28cfa5ce30e187b0d15b)), closes [#200](https://github.com/marcstraube/ansible-collection-common/issues/200)
* **php:** support AppStream and Remi on RHEL via php_redhat_repo ([#191](https://github.com/marcstraube/ansible-collection-common/issues/191)) ([6153132](https://github.com/marcstraube/ansible-collection-common/commit/615313277717829c4b7966b216829c8d6338bb79))
* **resolved:** skip service and symlink tasks when systemd-resolved unit is missing ([#185](https://github.com/marcstraube/ansible-collection-common/issues/185)) ([b38bab9](https://github.com/marcstraube/ansible-collection-common/commit/b38bab95cda9237038ce597a24b513c7dd20a77b))
* **templates:** make ansible_managed reference safe under ansible-core 2.23+ ([#205](https://github.com/marcstraube/ansible-collection-common/issues/205)) ([abb94d5](https://github.com/marcstraube/ansible-collection-common/commit/abb94d5f91e74399ea2aa13fc0b3fc18cc1e455b)), closes [#131](https://github.com/marcstraube/ansible-collection-common/issues/131)


### Documentation

* **migration:** adopt persistent Unreleased section (Keep a Changelog) ([#203](https://github.com/marcstraube/ansible-collection-common/issues/203)) ([e22afbf](https://github.com/marcstraube/ansible-collection-common/commit/e22afbfb68b828c2d0793ad63b56cfabc801a983)), closes [#162](https://github.com/marcstraube/ansible-collection-common/issues/162)
* **migration:** scope MIGRATION.md to actionable breaking changes only ([#216](https://github.com/marcstraube/ansible-collection-common/issues/216)) ([02c487f](https://github.com/marcstraube/ansible-collection-common/commit/02c487f05facf0c131a96524d84ff296b7316511)), closes [#215](https://github.com/marcstraube/ansible-collection-common/issues/215)

## [2.0.0](https://github.com/marcstraube/ansible-collection-common/compare/v1.2.0...v2.0.0) (2026-05-22)


### ⚠ BREAKING CHANGES

* bump ansible-core minimum to 2.19 + forward-compat to 2.21 ([#174](https://github.com/marcstraube/ansible-collection-common/issues/174))
* **networkmanager:** add generic settings dict for nmcli-native passthrough ([#156](https://github.com/marcstraube/ansible-collection-common/issues/156))
* **shell:** migrate shell role from marcstraube.desktop ([#137](https://github.com/marcstraube/ansible-collection-common/issues/137))
* **firejail:** disable by default + add cleanup path ([#118](https://github.com/marcstraube/ansible-collection-common/issues/118))
* **networkmanager:** remove toggle_wifi and WiFi auto-toggle dispatcher ([#114](https://github.com/marcstraube/ansible-collection-common/issues/114))
* Add NM-managed WireGuard mode for clients ([#102](https://github.com/marcstraube/ansible-collection-common/issues/102))
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
* **ci:** add drift-detection workflow for Layer-1-no-op'd packages ([#171](https://github.com/marcstraube/ansible-collection-common/issues/171)) ([27e47a5](https://github.com/marcstraube/ansible-collection-common/commit/27e47a5db749dfeea2313291466e6a4a56e237a7)), closes [#169](https://github.com/marcstraube/ansible-collection-common/issues/169)
* **networkmanager:** add generic settings dict for nmcli-native passthrough ([#156](https://github.com/marcstraube/ansible-collection-common/issues/156)) ([8a56a9f](https://github.com/marcstraube/ansible-collection-common/commit/8a56a9f540a9d33153db4cdf26232b031fb3710d)), closes [#153](https://github.com/marcstraube/ansible-collection-common/issues/153)
* **networkmanager:** assert UUIDs collected for all configured connections ([#124](https://github.com/marcstraube/ansible-collection-common/issues/124)) ([8b6a2ff](https://github.com/marcstraube/ansible-collection-common/commit/8b6a2ffbb7922dec14477761ea3381ad4e6d56e4)), closes [#121](https://github.com/marcstraube/ansible-collection-common/issues/121)
* **playbooks:** Add system_update task file for Arch partial-upgrade safety ([#110](https://github.com/marcstraube/ansible-collection-common/issues/110)) ([22fc70e](https://github.com/marcstraube/ansible-collection-common/commit/22fc70e009fd0d9a5badeedb34abea95d6be3adc))
* **shell:** add atuin, ble.sh, and bash-preexec toggles ([#146](https://github.com/marcstraube/ansible-collection-common/issues/146)) ([9368c2c](https://github.com/marcstraube/ansible-collection-common/commit/9368c2cc8f8afc0391d728013b3c19b6c03eb2aa)), closes [#133](https://github.com/marcstraube/ansible-collection-common/issues/133)
* **shell:** include fish in shell_zoxide_shells default ([#150](https://github.com/marcstraube/ansible-collection-common/issues/150)) ([c524f5f](https://github.com/marcstraube/ansible-collection-common/commit/c524f5fb46f482c77c9ae41a4fcf9b7afe2e152f)), closes [#149](https://github.com/marcstraube/ansible-collection-common/issues/149)
* **shell:** migrate shell role from marcstraube.desktop ([#137](https://github.com/marcstraube/ansible-collection-common/issues/137)) ([31d22f6](https://github.com/marcstraube/ansible-collection-common/commit/31d22f6f22fe113dd5eb6ec01ea7b82c926f37d0)), closes [#132](https://github.com/marcstraube/ansible-collection-common/issues/132)
* **utils:** add dust, duf, procs, age, sops toggles ([#145](https://github.com/marcstraube/ansible-collection-common/issues/145)) ([803a5bb](https://github.com/marcstraube/ansible-collection-common/commit/803a5bbca02c9cc7d9523356f53ea5f9db2efea9)), closes [#122](https://github.com/marcstraube/ansible-collection-common/issues/122)
* **utils:** add network diagnostics toggles ([#155](https://github.com/marcstraube/ansible-collection-common/issues/155)) ([9a40b4e](https://github.com/marcstraube/ansible-collection-common/commit/9a40b4e44f890ed57c02443184e189565227c4f2)), closes [#152](https://github.com/marcstraube/ansible-collection-common/issues/152)


### Bug Fixes

* Add --all argument to HSM sign script in apparmor ([#76](https://github.com/marcstraube/ansible-collection-common/issues/76)) ([efa068b](https://github.com/marcstraube/ansible-collection-common/commit/efa068bce9692a6a323e9d43698cc9a73ee8389b))
* Add missing fontconfig variable to fonts RedHat-9.yml ([#94](https://github.com/marcstraube/ansible-collection-common/issues/94)) ([42e612c](https://github.com/marcstraube/ansible-collection-common/commit/42e612c1c62437c8aac31624e49644ffd0b980e7))
* Add wait_for_connection after Docker service start ([#101](https://github.com/marcstraube/ansible-collection-common/issues/101)) ([b993730](https://github.com/marcstraube/ansible-collection-common/commit/b993730e8bee5e9d846697b80d2f4f53790c7392))
* AIDE exclude BTRFS snapshots and disable /home monitoring by default ([#108](https://github.com/marcstraube/ansible-collection-common/issues/108)) ([052be48](https://github.com/marcstraube/ansible-collection-common/commit/052be4866bb0b69e7a7127d6bc110cde00cef812))
* **aide:** empty package on Arch (Layer-1 no-op) instead of continue-on-error ([#170](https://github.com/marcstraube/ansible-collection-common/issues/170)) ([2e91d61](https://github.com/marcstraube/ansible-collection-common/commit/2e91d613b9206eee7b9df8d6c267ac46da320d70)), closes [#168](https://github.com/marcstraube/ansible-collection-common/issues/168)
* Auto-detect /tmp mount type in hardening role ([#87](https://github.com/marcstraube/ansible-collection-common/issues/87)) ([681d4ba](https://github.com/marcstraube/ansible-collection-common/commit/681d4ba603997cc6f8860dd939bdae7ba19366e0))
* **base-system:** include shell role in base_system.yml playbook ([#138](https://github.com/marcstraube/ansible-collection-common/issues/138)) ([db367bb](https://github.com/marcstraube/ansible-collection-common/commit/db367bb14d98325b5e4959887b8b2c5516d59779)), closes [#132](https://github.com/marcstraube/ansible-collection-common/issues/132)
* BTRFS NoCOW tasks fail on missing directories ([#78](https://github.com/marcstraube/ansible-collection-common/issues/78)) ([33dcf5d](https://github.com/marcstraube/ansible-collection-common/commit/33dcf5d7225e18b13626c303700fb2a4920f1990))
* **ci:** locate release PR via label instead of action output ([#167](https://github.com/marcstraube/ansible-collection-common/issues/167)) ([78d00b0](https://github.com/marcstraube/ansible-collection-common/commit/78d00b058cee5b1248c9aebbfe30d8e5dc99d303)), closes [#166](https://github.com/marcstraube/ansible-collection-common/issues/166)
* **ci:** provision aur_builder user in drift probe before sudo ([#172](https://github.com/marcstraube/ansible-collection-common/issues/172)) ([97a13a7](https://github.com/marcstraube/ansible-collection-common/commit/97a13a760cd17f754d53219a8d1c207a0807cc5a))
* Correct README titles to collection FQCN format ([#51](https://github.com/marcstraube/ansible-collection-common/issues/51)) ([89a8885](https://github.com/marcstraube/ansible-collection-common/commit/89a8885d359b6648bd7971c21ef8e15385a07ca1))
* Install AppArmor extra profiles before enforcing ([#106](https://github.com/marcstraube/ansible-collection-common/issues/106)) ([7e2e4bf](https://github.com/marcstraube/ansible-collection-common/commit/7e2e4bfc29cfa01348513ff19c3dceb58aad8ab9))
* Make BTRFS NoCOW tasks idempotent across 8 roles ([#104](https://github.com/marcstraube/ansible-collection-common/issues/104)) ([993e345](https://github.com/marcstraube/ansible-collection-common/commit/993e3450b374d773739a4023d3b40edb20c4deeb))
* **molecule:** drop play-level become: true from 34 prepare.yml files ([#148](https://github.com/marcstraube/ansible-collection-common/issues/148)) ([e09271b](https://github.com/marcstraube/ansible-collection-common/commit/e09271b851be7e91ba215c4be8d8c4980344b9d2)), closes [#147](https://github.com/marcstraube/ansible-collection-common/issues/147)
* **networkmanager:** classify SMB dispatcher depends_on by inventory data ([#123](https://github.com/marcstraube/ansible-collection-common/issues/123)) ([a162666](https://github.com/marcstraube/ansible-collection-common/commit/a1626664c5c66306744dd41f4e27c1a5bc45fc71)), closes [#120](https://github.com/marcstraube/ansible-collection-common/issues/120) [#121](https://github.com/marcstraube/ansible-collection-common/issues/121)
* **networkmanager:** reapply NetworkManager device after connection modify ([#154](https://github.com/marcstraube/ansible-collection-common/issues/154)) ([a6e6be0](https://github.com/marcstraube/ansible-collection-common/commit/a6e6be05235d67f879dd3c3f3cb92e64e8dbea8a)), closes [#151](https://github.com/marcstraube/ansible-collection-common/issues/151)
* PHP role auto-version and Arch extension mapping ([#80](https://github.com/marcstraube/ansible-collection-common/issues/80)) ([cc0ef93](https://github.com/marcstraube/ansible-collection-common/commit/cc0ef930f1a3f8db3108c609ad2748fcb75f4c62))
* **podman:** skip docker socket symlink when real Docker is installed ([#139](https://github.com/marcstraube/ansible-collection-common/issues/139)) ([d4a725e](https://github.com/marcstraube/ansible-collection-common/commit/d4a725ec3fb0dc804f5d82cd2fed30cbf84869d3)), closes [#130](https://github.com/marcstraube/ansible-collection-common/issues/130)
* read-only command tasks must survive --check (check_mode: false) ([#129](https://github.com/marcstraube/ansible-collection-common/issues/129)) ([e327bac](https://github.com/marcstraube/ansible-collection-common/commit/e327bac2804c6652aff8c07394b5081f5bd36ad4)), closes [#128](https://github.com/marcstraube/ansible-collection-common/issues/128)
* Remove _ansible_managed from JSON templates ([#72](https://github.com/marcstraube/ansible-collection-common/issues/72)) ([f89fb1a](https://github.com/marcstraube/ansible-collection-common/commit/f89fb1a136a3658add0a0fc08d7f50f45d7959ac))
* Reset FPM service name for system PHP in configure phase ([#85](https://github.com/marcstraube/ansible-collection-common/issues/85)) ([62e71c5](https://github.com/marcstraube/ansible-collection-common/commit/62e71c5475d5695d4ec10cb5c8045d95dc8f3573))
* **shell:** source zoxide init in user rc files ([#142](https://github.com/marcstraube/ansible-collection-common/issues/142)) ([49f15bb](https://github.com/marcstraube/ansible-collection-common/commit/49f15bbe991675b311037997c1d5c13fa546e32b)), closes [#134](https://github.com/marcstraube/ansible-collection-common/issues/134)
* Skip podman-docker when real Docker is installed ([#77](https://github.com/marcstraube/ansible-collection-common/issues/77)) ([c783a10](https://github.com/marcstraube/ansible-collection-common/commit/c783a10c24b8a51bc7a16efac6b64338307c2e76))
* Use official repo for python-podman on Arch ([#73](https://github.com/marcstraube/ansible-collection-common/issues/73)) ([5c881e7](https://github.com/marcstraube/ansible-collection-common/commit/5c881e7e50216a131452c8804208661a0a23299d))


### Code Refactoring

* Add NM-managed WireGuard mode for clients ([#102](https://github.com/marcstraube/ansible-collection-common/issues/102)) ([9865275](https://github.com/marcstraube/ansible-collection-common/commit/9865275ac55123fe14d471f21f05ea2b7aff6c83))
* Add playbook task files for infra delegation ([#71](https://github.com/marcstraube/ansible-collection-common/issues/71)) ([9a6e459](https://github.com/marcstraube/ansible-collection-common/commit/9a6e4590a23ee8215d554e502a52357d6390ba15))
* Change user_config_mode defaults to initial ([#62](https://github.com/marcstraube/ansible-collection-common/issues/62)) ([386e648](https://github.com/marcstraube/ansible-collection-common/commit/386e64848df75e31388f53363ffec0d07c571bb4))
* **firejail:** disable by default + add cleanup path ([#118](https://github.com/marcstraube/ansible-collection-common/issues/118)) ([64c0f10](https://github.com/marcstraube/ansible-collection-common/commit/64c0f1078c3bea3685d8d3c275e523a751720e8d)), closes [#117](https://github.com/marcstraube/ansible-collection-common/issues/117)
* Merge security and backup tasks into base_system.yml ([#96](https://github.com/marcstraube/ansible-collection-common/issues/96)) ([f8770dc](https://github.com/marcstraube/ansible-collection-common/commit/f8770dc230a684e81abf15e00e0042def1bddd53))
* **networkmanager:** drop toggle_wifi cleanup task ([#116](https://github.com/marcstraube/ansible-collection-common/issues/116)) ([f00bb54](https://github.com/marcstraube/ansible-collection-common/commit/f00bb54af9bf44262559ff53144a0acae5a36c8a))
* **networkmanager:** remove toggle_wifi and WiFi auto-toggle dispatcher ([#114](https://github.com/marcstraube/ansible-collection-common/issues/114)) ([4be30a0](https://github.com/marcstraube/ansible-collection-common/commit/4be30a087569117a91eff85d7de6b188152ed47e))
* **php:** Move Sury repo setup from package_management to php role ([#45](https://github.com/marcstraube/ansible-collection-common/issues/45)) ([5616f89](https://github.com/marcstraube/ansible-collection-common/commit/5616f89f46e655005abc4f60cf185c30ddfa61c2))
* Remove deprecated variable renames ([#60](https://github.com/marcstraube/ansible-collection-common/issues/60)) ([61d7c3a](https://github.com/marcstraube/ansible-collection-common/commit/61d7c3a1e1af72dabb00ee0fdb3d588dbad79772))
* Remove multiplexer variables from utils role ([#92](https://github.com/marcstraube/ansible-collection-common/issues/92)) ([c9b0654](https://github.com/marcstraube/ansible-collection-common/commit/c9b06542aef07484dfd4ad7a3d524d3367120801))
* Remove software-specific repos from package_management ([#61](https://github.com/marcstraube/ansible-collection-common/issues/61)) ([e220163](https://github.com/marcstraube/ansible-collection-common/commit/e22016373df39dc6490e20ff60f8b4b782617f58))
* Rename base.yml to base_system.yml and remove networking.yml ([#56](https://github.com/marcstraube/ansible-collection-common/issues/56)) ([815e1c0](https://github.com/marcstraube/ansible-collection-common/commit/815e1c03d98576a0ee9d028c9bc43fadc453e1e1))
* Update backup.yml to default(true) ([#58](https://github.com/marcstraube/ansible-collection-common/issues/58)) ([be822df](https://github.com/marcstraube/ansible-collection-common/commit/be822dfcdec4b591ee8a94d970bc920d54d4a2fe))
* Update security.yml to default(true) and full role coverage ([#57](https://github.com/marcstraube/ansible-collection-common/issues/57)) ([25ca9dc](https://github.com/marcstraube/ansible-collection-common/commit/25ca9dc11f998ac25e3bbed13ce1755b2d85a872))
* WireGuard role as standalone tunnel manager ([#83](https://github.com/marcstraube/ansible-collection-common/issues/83)) ([#88](https://github.com/marcstraube/ansible-collection-common/issues/88)) ([b356db0](https://github.com/marcstraube/ansible-collection-common/commit/b356db0c05ff677b62667b07761f38744b061265))
* **wireguard:** manage NM connection profile via ini_file ([#125](https://github.com/marcstraube/ansible-collection-common/issues/125)) ([102e64c](https://github.com/marcstraube/ansible-collection-common/commit/102e64cba45f2d1ef0dfdaf5fa02c783fc2e1162)), closes [#119](https://github.com/marcstraube/ansible-collection-common/issues/119)


### Documentation

* Add References sections to all role READMEs ([#63](https://github.com/marcstraube/ansible-collection-common/issues/63)) ([8c747ed](https://github.com/marcstraube/ansible-collection-common/commit/8c747edd5c63f7834e8077cfdec9559a111c4315))
* backfill MIGRATION.md with v2.0.0 breaking-change sections ([#160](https://github.com/marcstraube/ansible-collection-common/issues/160)) ([aa9c8f6](https://github.com/marcstraube/ansible-collection-common/commit/aa9c8f67a06c4460ae7dc71ab43ff49d8c648ca7)), closes [#157](https://github.com/marcstraube/ansible-collection-common/issues/157)
* **podman:** Document userns modes and per-container override ([#41](https://github.com/marcstraube/ansible-collection-common/issues/41)) ([cdfc659](https://github.com/marcstraube/ansible-collection-common/commit/cdfc6590d46d368b457f385fd9ea4d0b281038e6))
* Update README role count and fix pre-commit performance ([#40](https://github.com/marcstraube/ansible-collection-common/issues/40)) ([eca6d8d](https://github.com/marcstraube/ansible-collection-common/commit/eca6d8d3b6bf4bfab675cbc4e266a8e7ddffa7d5))


### Miscellaneous

* bump ansible-core minimum to 2.19 + forward-compat to 2.21 ([#174](https://github.com/marcstraube/ansible-collection-common/issues/174)) ([333ed99](https://github.com/marcstraube/ansible-collection-common/commit/333ed99078ca9643b5832bb9ceacfd3c86b9a669)), closes [#173](https://github.com/marcstraube/ansible-collection-common/issues/173)

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
