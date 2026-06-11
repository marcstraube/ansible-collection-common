# Migration Guide

Upgrade guide for users of `marcstraube.common`. Each release with
breaking changes adds a section here with before/after inventory
snippets. For the full list of changes per release, see
[CHANGELOG.md](CHANGELOG.md).

Following the [Keep a Changelog](https://keepachangelog.com/) convention,
the file always carries a persistent `## Unreleased` section at the top.
Breaking-change PRs append their entries under that section; the release
workflow dates it to `## v<X.Y.Z> - YYYY-MM-DD` at release time and
inserts a fresh empty `## Unreleased` above it. Do not delete the
heading or rename it to a concrete version — the workflow handles that.

## Unreleased

### `php` — RHEL default switched from Remi to AppStream

Previously `roles/php/vars/RedHat.yml` shipped unconditional Remi-SCL
package patterns (`php<XX>-php-cli`, `/etc/opt/remi/php<XX>/...`), but
the role did not deploy the Remi repository. On any RHEL-family host
without Remi pre-installed, the role failed with
`No package php<XX>-php-cli available`.

v2.0.1 introduces `php_redhat_repo` (`'appstream'` default, `'remi'`
opt-in). AppStream uses the distro module streams and unversioned
package names. Remi keeps the versioned `php<XX>-php-*` packages and
`/etc/opt/remi/` paths for side-by-side installs.

The Remi repository itself is not managed by the `php` role — it is
multi-purpose (PHP, MariaDB, Redis, …) and lives in
`marcstraube.common.package_management` under `dnf_remi_enabled`. The
`php` role asserts the repo is on the host when
`php_redhat_repo: 'remi'` is requested and fails with a hint otherwise.

AppStream's `dnf module` streams are exclusive, so on `appstream` the
role only accepts a single entry in `php_versions` (multi-version
side-by-side requires `remi`). The role also asserts that the
requested PHP version matches the active stream and fails with a hint
otherwise.

#### Required action

Hosts that already had Remi installed and relied on the previous
behaviour need both flags:

```yaml
# inventory: group_vars / host_vars
dnf_remi_enabled: true            # marcstraube.common.package_management
php_redhat_repo: 'remi'           # marcstraube.common.php
```

Hosts on AppStream get the default and need no inventory change, but
the active module stream must match `php_versions[0].version`. To
switch the active stream:

```bash
sudo dnf module reset -y php
sudo dnf module enable -y php:8.2
```

### `hardening` — `/tmp` is no longer auto-converted to tmpfs

The `/tmp` entry in `hardening_fs_mounts` is now only applied when the
current `/tmp` is already a tmpfs. On hosts where `/tmp` lives on the
root filesystem, on a BTRFS subvolume, or on another non-tmpfs mount,
the entry is skipped instead of silently masking the existing `/tmp`
contents with a fresh tmpfs.

The previous default could break running services (`noexec` on `/tmp`
prevents tools like `npm` from executing scripts there) and would
hide files already on `/tmp` from any process that opened them after
the remount.

#### Required action

Hosts that should still get `/tmp` converted to tmpfs — typically
fresh-install or installer workflows where `/tmp` is known to be
empty — need to opt in:

```yaml
# inventory: group_vars / host_vars
hardening_fs_tmp_force_tmpfs: true
```

Hosts where `/tmp` is already tmpfs (e.g. Arch Linux with the
default systemd `tmp.mount`) are unaffected — the hardened mount
options are still applied.

## v2.0.0 - 2026-05-22

### Minimum ansible-core bumped from 2.17 to 2.19 (#173)

v2.0.0 unifies and raises the supported ansible-core floor. Previous
v1.x state was inconsistent across surfaces (`meta/runtime.yml` declared
`>=2.15.0` while READMEs and `galaxy.yml` claimed 2.17+); v2.0.0 aligns
everything on **2.19.0** and bumps the CI forward-compat job to track
2.21 (latest stable) so it stays two versions above the floor.

The floor stops at 2.19 rather than 2.20 because ansible-lint v26.4.0
(latest at v2.0.0 release time) does not yet accept `>=2.20.0` in
`meta/runtime.yml` — its `meta-runtime` rule only knows about 2.15
through 2.19. Once ansible-lint catches up, a follow-up minor release
will bump the floor to 2.20.

The 2.17 → 2.19 step does not change the install path for users on
RHEL 9 or Debian Bookworm — those distros' system `ansible-core` is
older than 2.17 anyway, so a `pip install --user ansible-core` (or
`pipx`) is the established workflow there. The bump just makes the
actually-supported floor honest.

#### Required action

Upgrade `ansible-core` to 2.19 or newer before installing this
collection's v2.0.0 release:

```bash
# pip (system Python or venv)
pip install --upgrade 'ansible-core>=2.19,<2.22'

# pipx
pipx upgrade --pip-args='--upgrade' ansible-core
```

Distro packages currently shipping `ansible-core >= 2.19`:

- Arch Linux: `extra/ansible-core`
- Debian Trixie: `main/ansible-core`
- EPEL 10: `epel/ansible-core`
- Fedora 41+: `fedora/ansible-core`

If you pin `ansible-core` in a requirements file, set:

```text
ansible-core>=2.19,<2.22
```

### Deprecated v1.x variable renames removed (#60)

27 roles had v1.x variable names kept as deprecated aliases throughout
the v1.2.x series. All of those aliases (and their `set_fact`
fallback handlers) are removed in v2.0.0. Inventories that still use
the old names must be updated to the new `_enabled`-suffixed forms
listed below.

#### Before

```yaml
# Old v1.x inventory variable names
aide_monitor_binaries: true
ansible_tools_install_molecule: true
openssh_server_enabled: true
utils_fun_cowsay: true
```

#### After

```yaml
# New v2.0.0 variable names
aide_monitor_binaries_enabled: true
ansible_tools_molecule_enabled: true
openssh_enabled: true
utils_fun_cowsay_enabled: true
```

#### Complete rename table by role

##### `aide`

| Old name                    | New name                            |
|-----------------------------|-------------------------------------|
| `aide_monitor_binaries`     | `aide_monitor_binaries_enabled`     |
| `aide_monitor_libraries`    | `aide_monitor_libraries_enabled`    |
| `aide_monitor_boot`         | `aide_monitor_boot_enabled`         |
| `aide_monitor_configs`      | `aide_monitor_configs_enabled`      |
| `aide_monitor_system_dirs`  | `aide_monitor_system_dirs_enabled`  |
| `aide_monitor_home_dirs`    | `aide_monitor_home_dirs_enabled`    |
| `aide_monitor_logs`         | `aide_monitor_logs_enabled`         |
| `aide_init_db`              | `aide_init_db_enabled`              |
| `aide_force_init`           | `aide_force_init_enabled`           |

##### `ansible`

| Old name                         | New name                         |
|----------------------------------|----------------------------------|
| `ansible_tools_install_ansible`  | `ansible_tools_ansible_enabled`  |
| `ansible_tools_install_molecule` | `ansible_tools_molecule_enabled` |
| `ansible_tools_install_lint`     | `ansible_tools_lint_enabled`     |

##### `apparmor`

| Old name                  | New name                          |
|---------------------------|-----------------------------------|
| `apparmor_extra_profiles` | `apparmor_extra_profiles_enabled` |
| `apparmor_notify_package` | `apparmor_notify_enabled`         |

##### `auditd`

| Old name                  | New name                          |
|---------------------------|-----------------------------------|
| `auditd_pci_dss`          | `auditd_pci_dss_enabled`          |
| `auditd_hipaa`            | `auditd_hipaa_enabled`            |
| `auditd_nist`             | `auditd_nist_enabled`             |
| `auditd_stig`             | `auditd_stig_enabled`             |
| `auditd_time_change`      | `auditd_time_change_enabled`      |
| `auditd_identity_change`  | `auditd_identity_change_enabled`  |
| `auditd_network_change`   | `auditd_network_change_enabled`   |
| `auditd_system_locale`    | `auditd_system_locale_enabled`    |
| `auditd_mac_policy`       | `auditd_mac_policy_enabled`       |
| `auditd_logins`           | `auditd_logins_enabled`           |
| `auditd_session`          | `auditd_session_enabled`          |
| `auditd_perm_mod`         | `auditd_perm_mod_enabled`         |
| `auditd_access`           | `auditd_access_enabled`           |
| `auditd_delete`           | `auditd_delete_enabled`           |
| `auditd_scope`            | `auditd_scope_enabled`            |
| `auditd_actions`          | `auditd_actions_enabled`          |
| `auditd_modules`          | `auditd_modules_enabled`          |
| `auditd_remote_logging`   | `auditd_remote_logging_enabled`   |
| `auditd_plugin_syslog`    | `auditd_plugin_syslog_enabled`    |

##### `avahi`

| Old name          | New name                  |
|-------------------|---------------------------|
| `avahi_nss_mdns`  | `avahi_nss_mdns_enabled`  |

##### `base`

| Old name       | New name              |
|----------------|-----------------------|
| `base_dialog`  | `base_dialog_enabled` |

##### `clamav`

| Old name                       | New name                               |
|--------------------------------|----------------------------------------|
| `clamav_postfix_integration`   | `clamav_postfix_integration_enabled`   |
| `clamav_rspamd_integration`    | `clamav_rspamd_integration_enabled`    |

##### `editors`

| Old name                        | New name                         |
|---------------------------------|----------------------------------|
| `editors_nano`                  | `editors_nano_enabled`           |
| `editors_vim`                   | `editors_vim_enabled`            |
| `editors_neovim`                | `editors_neovim_enabled`         |
| `editors_nano_config_manage`    | `editors_nano_config_enabled`    |
| `editors_vim_config_manage`     | `editors_vim_config_enabled`     |
| `editors_neovim_config_manage`  | `editors_neovim_config_enabled`  |
| `editors_neovim_binary_install` | `editors_neovim_binary_enabled`  |

##### `energy_management`

| Old name                              | New name                                      |
|---------------------------------------|-----------------------------------------------|
| `energy_management_battery_tp_smapi`  | `energy_management_battery_tp_smapi_enabled`  |

##### `fail2ban`

| Old name                          | New name                                  |
|-----------------------------------|-------------------------------------------|
| `fail2ban_bantime_increment`      | `fail2ban_bantime_increment_enabled`      |
| `fail2ban_bantime_overalljails`   | `fail2ban_bantime_overalljails_enabled`   |

##### `firejail`

| Old name                | New name                        |
|-------------------------|---------------------------------|
| `firejail_apparmor`     | `firejail_apparmor_enabled`     |
| `firejail_pacman_hook`  | `firejail_pacman_hook_enabled`  |

##### `fonts`

| Old name                       | New name                               |
|--------------------------------|----------------------------------------|
| `fonts_base`                   | `fonts_base_enabled`                   |
| `fonts_dejavu`                 | `fonts_dejavu_enabled`                 |
| `fonts_liberation`             | `fonts_liberation_enabled`             |
| `fonts_noto`                   | `fonts_noto_enabled`                   |
| `fonts_noto_emoji`             | `fonts_noto_emoji_enabled`             |
| `fonts_noto_cjk`               | `fonts_noto_cjk_enabled`               |
| `fonts_coding`                 | `fonts_coding_enabled`                 |
| `fonts_nerd`                   | `fonts_nerd_enabled`                   |
| `fonts_fontconfig_antialias`   | `fonts_fontconfig_antialias_enabled`   |
| `fonts_rebuild_cache`          | `fonts_rebuild_cache_enabled`          |

##### `gnupg`

| Old name                  | New name                  |
|---------------------------|---------------------------|
| `gnupg_install_gui`       | `gnupg_gui_enabled`       |
| `gnupg_install_tools`     | `gnupg_tools_enabled`     |
| `gnupg_agent_enable_ssh`  | `gnupg_agent_ssh_enabled` |

##### `graphics`

| Old name                                  | New name                                          |
|-------------------------------------------|---------------------------------------------------|
| `graphics_intel_mesa`                     | `graphics_intel_mesa_enabled`                     |
| `graphics_intel_vulkan`                   | `graphics_intel_vulkan_enabled`                   |
| `graphics_intel_vaapi`                    | `graphics_intel_vaapi_enabled`                    |
| `graphics_intel_guc_huc`                  | `graphics_intel_guc_huc_enabled`                  |
| `graphics_intel_32bit`                    | `graphics_intel_32bit_enabled`                    |
| `graphics_amd_mesa`                       | `graphics_amd_mesa_enabled`                       |
| `graphics_amd_vulkan`                     | `graphics_amd_vulkan_enabled`                     |
| `graphics_amd_32bit`                      | `graphics_amd_32bit_enabled`                      |
| `graphics_nvidia_32bit`                   | `graphics_nvidia_32bit_enabled`                   |
| `graphics_nvidia_cuda`                    | `graphics_nvidia_cuda_enabled`                    |
| `graphics_vulkan_tools`                   | `graphics_vulkan_tools_enabled`                   |
| `graphics_vaapi_tools`                    | `graphics_vaapi_tools_enabled`                    |
| `graphics_vdpau_tools`                    | `graphics_vdpau_tools_enabled`                    |
| `graphics_mesa_utils`                     | `graphics_mesa_utils_enabled`                     |
| `graphics_hybrid_switcheroo`              | `graphics_hybrid_switcheroo_enabled`              |
| `graphics_hybrid_prime_run`               | `graphics_hybrid_prime_run_enabled`               |
| `graphics_hybrid_nvidia_rtd3`             | `graphics_hybrid_nvidia_rtd3_enabled`             |
| `graphics_hybrid_nvidia_persistenced`     | `graphics_hybrid_nvidia_persistenced_enabled`     |

##### `hardening`

| Old name                                | New name                                        |
|-----------------------------------------|-------------------------------------------------|
| `hardening_kernel_perf_event_paranoid`  | `hardening_kernel_perf_event_paranoid_enabled`  |
| `hardening_fs_vartmp_bind`              | `hardening_fs_vartmp_bind_enabled`              |

##### `hardware_tokens`

| Old name                                   | New name                                       |
|--------------------------------------------|------------------------------------------------|
| `hardware_tokens_nitrokey`                 | `hardware_tokens_nitrokey_enabled`             |
| `hardware_tokens_nitrokey_app`             | `hardware_tokens_nitrokey_app_enabled`         |
| `hardware_tokens_yubikey`                  | `hardware_tokens_yubikey_enabled`              |
| `hardware_tokens_yubikey_manager`          | `hardware_tokens_yubikey_manager_enabled`      |
| `hardware_tokens_fido2`                    | `hardware_tokens_fido2_enabled`                |
| `hardware_tokens_smartcard`                | `hardware_tokens_smartcard_enabled`            |
| `hardware_tokens_opensc`                   | `hardware_tokens_opensc_enabled`               |
| `hardware_tokens_pcscd`                    | `hardware_tokens_pcscd_socket_enabled`         |
| `hardware_tokens_pcscd_enabled`            | `hardware_tokens_pcscd_socket_enabled`         |
| `hardware_tokens_pam_u2f`                  | `hardware_tokens_pam_u2f_enabled`              |
| `hardware_tokens_pam_u2f_pin`              | `hardware_tokens_pam_u2f_pin_enabled`          |
| `hardware_tokens_pam_u2f_nouserok`         | `hardware_tokens_pam_u2f_nouserok_enabled`     |
| `hardware_tokens_pam_u2f_system_auth`      | `hardware_tokens_pam_u2f_system_auth_enabled`  |
| `hardware_tokens_udev_rules`               | `hardware_tokens_udev_rules_enabled`           |
| `hardware_tokens_lock_on_removal`          | `hardware_tokens_lock_on_removal_enabled`      |

##### `multiplexer`

The multiplexer role was extracted from `utils` in v1.2.x. Variables
were renamed when the role split out.

| Old name (`utils` role)     | New name (`multiplexer` role)  |
|-----------------------------|--------------------------------|
| `utils_mux_tmux_enabled`    | `multiplexer_tmux_enabled`     |
| `utils_mux_zellij_enabled`  | `multiplexer_zellij_enabled`   |

##### `networkmanager`

| Old name                            | New name                          |
|-------------------------------------|-----------------------------------|
| `networkmanager_install_applet`     | `networkmanager_applet_enabled`   |
| `networkmanager_configure_timezone` | `networkmanager_timezone_enabled` |
| `networkmanager_polkit`             | `networkmanager_polkit_enabled`   |

##### `openssh`

| Old name                                | New name                                        |
|-----------------------------------------|-------------------------------------------------|
| `openssh_server_enabled`                | `openssh_enabled`                               |
| `openssh_server_state`                  | replaced by `openssh_service_enabled` (ternary) |
| `openssh_server_harden_moduli`          | `openssh_server_harden_moduli_enabled`          |
| `openssh_agent_global_service`          | `openssh_agent_global_service_enabled`          |
| `openssh_server_generate_sshfp_records` | `openssh_server_generate_sshfp_records_enabled` |
| `openssh_install_ssh_audit`             | `openssh_ssh_audit_enabled`                     |

`openssh_server_state` was previously a string (`started` / `stopped`).
Set `openssh_service_enabled: true` / `false` instead — the role uses a
ternary to manage the service state.

##### `package_management`

| Old name                          | New name                          |
|-----------------------------------|-----------------------------------|
| `pacman_testing_repos`            | `pacman_testing_repos_enabled`    |
| `pacman_multilib`                 | `pacman_multilib_enabled`         |
| `pacman_multilib_testing`         | `pacman_multilib_testing_enabled` |
| `makepkg_distcc`                  | `makepkg_distcc_enabled`          |
| `makepkg_ccache`                  | `makepkg_ccache_enabled`          |
| `makepkg_sccache`                 | `makepkg_sccache_enabled`         |
| `paccache_enabled`                | `paccache_timer_enabled`          |
| `pacman_install_rebuild_detector` | `pacman_rebuild_detector_enabled` |
| `pacman_install_pkgfile`          | `pacman_pkgfile_enabled`          |
| `pacman_install_informant`        | `pacman_informant_enabled`        |
| `apt_unattended_upgrades`         | `apt_unattended_upgrades_enabled` |
| `apt_install_apt_file`            | `apt_apt_file_enabled`            |
| `apt_install_needrestart`         | `apt_needrestart_enabled`         |
| `apt_install_debsums`             | `apt_debsums_enabled`             |
| `apt_install_debian_goodies`      | `apt_debian_goodies_enabled`      |
| `dnf_makecache_timer`             | `dnf_makecache_timer_enabled`     |
| `dnf_rpmfusion_free`              | `dnf_rpmfusion_free_enabled`      |
| `dnf_rpmfusion_nonfree`           | `dnf_rpmfusion_nonfree_enabled`   |
| `dnf_install_utils`               | `dnf_utils_enabled`               |
| `dnf_install_needs_restarting`    | `dnf_needs_restarting_enabled`    |
| `dnf_install_rpmbuild`            | `dnf_rpmbuild_enabled`            |
| `dnf_install_mock`                | `dnf_mock_enabled`                |

Software-specific repo variables (`apt_docker_repo`, `apt_postgresql_repo`,
`apt_php_sury_repo`, `apt_grafana_repo`, `apt_kubernetes_repo`,
`dnf_docker_repo`, `dnf_postgresql_repo`, `dnf_grafana_repo`,
`dnf_kubernetes_repo`) are removed entirely — see #61 below.

##### `pki`

| Old name              | New name                    |
|-----------------------|-----------------------------|
| `pki_configure_system`| `pki_system_config_enabled` |
| `pki_disable_legacy`  | `pki_disable_legacy_enabled`|
| `pki_create_dirs`     | `pki_create_dirs_enabled`   |

##### `podman`

| Old name                       | New name                                     |
|--------------------------------|----------------------------------------------|
| `podman_docker_alias`          | `podman_docker_alias_enabled`                |
| `podman_docker_socket_symlink` | `podman_docker_socket_symlink_enabled`       |
| `podman_btrfs_enabled`         | removed — BTRFS storage driver auto-detected |

##### `sudo`

| Old name                | New name                        |
|-------------------------|---------------------------------|
| `sudo_wheel_group`      | `sudo_wheel_group_enabled`      |
| `sudo_purge_sudoers_d`  | `sudo_purge_sudoers_d_enabled`  |

##### `sysctl`

| Old name                            | New name                                    |
|-------------------------------------|---------------------------------------------|
| `sysctl_profile_security`           | `sysctl_profile_security_enabled`           |
| `sysctl_profile_network_performance`| `sysctl_profile_network_performance_enabled`|
| `sysctl_profile_database`           | `sysctl_profile_database_enabled`           |
| `sysctl_profile_webserver`          | `sysctl_profile_webserver_enabled`          |
| `sysctl_profile_docker`             | `sysctl_profile_docker_enabled`             |

##### `unbound`

| Old name                      | New name                              |
|-------------------------------|---------------------------------------|
| `unbound_root_hints_update`   | `unbound_root_hints_update_enabled`   |
| `unbound_remote_control`      | `unbound_remote_control_enabled`      |

##### `utils`

| Old name                          | New name                                  |
|-----------------------------------|-------------------------------------------|
| `utils_monitor_htop`              | `utils_monitor_htop_enabled`              |
| `utils_monitor_btop`              | `utils_monitor_btop_enabled`              |
| `utils_monitor_bottom`            | `utils_monitor_bottom_enabled`            |
| `utils_monitor_atop`              | `utils_monitor_atop_enabled`              |
| `utils_mux_tmux`                  | use `multiplexer_tmux_enabled` (see #92)  |
| `utils_mux_zellij`                | use `multiplexer_zellij_enabled` (see #92)|
| `utils_mux_screen`                | `utils_mux_screen_enabled`                |
| `utils_transfer_rsync`            | `utils_transfer_rsync_enabled`            |
| `utils_search_ripgrep`            | `utils_search_ripgrep_enabled`            |
| `utils_search_fd`                 | `utils_search_fd_enabled`                 |
| `utils_fetch_fastfetch`           | `utils_fetch_fastfetch_enabled`           |
| `utils_fetch_hyfetch`             | `utils_fetch_hyfetch_enabled`             |
| `utils_fetch_archey4`             | `utils_fetch_archey4_enabled`             |
| `utils_fetch_cpufetch`            | `utils_fetch_cpufetch_enabled`            |
| `utils_fun_cowsay`                | `utils_fun_cowsay_enabled`                |
| `utils_fun_fortune`               | `utils_fun_fortune_enabled`               |
| `utils_fun_lolcat`                | `utils_fun_lolcat_enabled`                |
| `utils_fun_ponysay`               | `utils_fun_ponysay_enabled`               |
| `utils_fun_figlet`                | `utils_fun_figlet_enabled`                |
| `utils_fun_toilet`                | `utils_fun_toilet_enabled`                |
| `utils_fun_sl`                    | `utils_fun_sl_enabled`                    |
| `utils_fun_cmatrix`               | `utils_fun_cmatrix_enabled`               |
| `utils_fun_cbonsai`               | `utils_fun_cbonsai_enabled`               |
| `utils_fun_asciiquarium`          | `utils_fun_asciiquarium_enabled`          |
| `utils_fun_nyancat`               | `utils_fun_nyancat_enabled`               |
| `utils_fun_no_more_secrets`       | `utils_fun_no_more_secrets_enabled`       |
| `utils_fun_tty_clock`             | `utils_fun_tty_clock_enabled`             |
| `utils_fun_pipes`                 | `utils_fun_pipes_enabled`                 |
| `utils_fun_pokemon_colorscripts`  | `utils_fun_pokemon_colorscripts_enabled`  |
| `utils_fun_pokemonsay`            | `utils_fun_pokemonsay_enabled`            |
| `utils_fun_hollywood`             | `utils_fun_hollywood_enabled`             |

##### `wireguard`

| Old name                  | New name                                 |
|---------------------------|------------------------------------------|
| `wireguard_server_mode`   | removed — superseded by #83/#88 refactor |
| `wireguard_ip_forward`    | removed — superseded by #83/#88 refactor |
| `wireguard_client_mode`   | removed — superseded by #83/#88 refactor |
| `wireguard_generate_keys` | removed — superseded by #83/#88 refactor |
| `wireguard_saveconfig`    | removed — superseded by #83/#88 refactor |

The entire flat-variable WireGuard schema was replaced by the
`wireguard_interfaces` list. See the dedicated #83/#88 section below.

### `package_management` — software-specific repos removed (#61)

Software-specific repositories no longer belong in
`package_management`. They are now managed by the role that owns the
software (or by a role in `marcstraube.server`).

Removed: PostgreSQL PGDG, Grafana, Kubernetes, Sury PHP, Docker CE
repository tasks and variables across both `apt_*` and `dnf_*`
families. Multi-purpose distribution repos (EPEL, Remi, RPM Fusion,
ELRepo, CRB, COPR, custom) stay in `package_management`.

#### Before

```yaml
# Old: enabled software-specific repos via package_management
apt_docker_repo_enabled: true
apt_postgresql_repo_enabled: true
apt_grafana_repo_enabled: true
apt_kubernetes_repo_enabled: true
apt_sury_enabled: true

dnf_docker_repo_enabled: true
dnf_postgresql_repo_enabled: true
dnf_grafana_repo_enabled: true
dnf_kubernetes_repo_enabled: true
```

#### After

Each removed variable has a new owner that manages its repo
internally — drop the old variable from inventory and include the
role wherever the repo is needed.

- `apt_docker_repo_enabled` / `dnf_docker_repo_enabled` →
  `marcstraube.common.docker` (role manages its own repo).
- `apt_sury_enabled` → `marcstraube.common.php` (role manages its own
  repo).
- `apt_postgresql_repo_enabled` / `dnf_postgresql_repo_enabled` →
  `marcstraube.server` PostgreSQL role (see
  [ansible-collection-server#7](https://github.com/marcstraube/ansible-collection-server/issues/7)).
- `apt_kubernetes_repo_enabled` / `dnf_kubernetes_repo_enabled` →
  `marcstraube.server` Kubernetes role (see
  [ansible-collection-server#8](https://github.com/marcstraube/ansible-collection-server/issues/8)).
- `apt_grafana_repo_enabled` / `dnf_grafana_repo_enabled` →
  `marcstraube.server.grafana` (already manages its own repo).

### `user_config_mode` defaults changed from `managed` to `initial` (#62)

Three roles that ship per-user dotfiles now default to deploying them
once on first run and leaving them owned by the user afterwards. Set
the variable to `'managed'` in inventory if continuous role enforcement
is desired.

| Variable                       | v1.x default | v2.0.0 default |
|--------------------------------|--------------|----------------|
| `editors_user_config_mode`     | `'managed'`  | `'initial'`    |
| `gnupg_user_config_mode`       | `'managed'`  | `'initial'`    |
| `makepkg_user_config_mode`     | `'managed'`  | `'initial'`    |

`multiplexer_user_config_mode` already defaulted to `'initial'` — no
change.

#### Before

```yaml
# v1.x: role would re-deploy these dotfiles on every run
editors_user_config_mode: 'managed'
gnupg_user_config_mode: 'managed'
makepkg_user_config_mode: 'managed'
```

#### After

```yaml
# v2.0.0 default: dotfiles deployed once, then user-owned.
# Override only if you need ongoing enforcement.
editors_user_config_mode: 'managed'  # opt-in to old behavior
```

### `wireguard` — standalone tunnel manager with `wireguard_interfaces` list (#83/#88)

The WireGuard role no longer manages a single flat tunnel. Multi-interface
support is provided through the new `wireguard_interfaces` list, with
each entry describing a complete tunnel (server or client mode, peers,
keys, firewall, lifecycle scripts). All `wireguard_*` flat variables
are removed.

NetworkManager no longer manages WireGuard connections — the
`networkmanager_connections.wireguard` dict is removed (clients can use
the dedicated NM-managed mode added in #102 instead). NM dispatcher
scripts now call `systemctl` (wg-quick) rather than `nmcli` for VPN
control.

Additionally, `networkmanager_vpn_autoconnect.connection` was renamed
to `networkmanager_vpn_autoconnect.interface` (now references a
WireGuard interface name from `wireguard_interfaces`, not a NM
connection profile).

#### Before

```yaml
# v1.x: single-tunnel flat schema
wireguard_interface: 'wg0'
wireguard_address: '10.100.0.1/24'
wireguard_port: 51820
wireguard_private_key: ''
wireguard_dns:
  - '10.100.0.1'
wireguard_server_mode_enabled: true
wireguard_ip_forward_enabled: true
wireguard_peers:
  - name: 'laptop'
    public_key: 'abc123...'
    allowed_ips: '10.100.0.2/32'

# NM-managed WireGuard via networkmanager_connections
networkmanager_connections:
  wireguard:
    "Work VPN":
      uuid: '...'
      private_key: '...'
      # ...

networkmanager_vpn_autoconnect:
  connection: 'Work VPN'   # NM connection name
  # ...
```

#### After

```yaml
# v2.0.0: list of interfaces, each a standalone tunnel
wireguard_interfaces:
  - name: wg0
    address: '10.100.0.1/24'
    port: 51820
    mode: server          # or 'client', or 'nm' (see #102)
    ip_forward: true
    generate_keys: true
    peers:
      - name: 'laptop'
        public_key: 'abc123...'
        allowed_ips: '10.100.0.2/32'
        persistent_keepalive: 25

# networkmanager_connections.wireguard removed entirely
networkmanager_connections:
  ethernet: { ... }
  wifi: { ... }
  # no 'wireguard:' key any more

networkmanager_vpn_autoconnect:
  interface: 'wg0'   # WireGuard interface name from wireguard_interfaces
  # ...
```

### `utils` — multiplexer variables moved to dedicated role (#92)

The `tmux` and `zellij` toggles have moved out of `utils` into the
dedicated `multiplexer` role (which also gained a `screen` toggle).
Three legacy variables in `utils` are removed.

| Removed from `utils`         | Replacement                             |
|------------------------------|-----------------------------------------|
| `utils_mux_tmux_enabled`     | `multiplexer_tmux_enabled` (new role)   |
| `utils_mux_zellij_enabled`   | `multiplexer_zellij_enabled` (new role) |
| `utils_mux_screen_enabled`   | `multiplexer_screen_enabled` (new role) |

#### Before

```yaml
# v1.x: multiplexer toggles inside utils
utils_mux_tmux_enabled: true
utils_mux_zellij_enabled: true
utils_mux_screen_enabled: false
```

#### After

```yaml
# v2.0.0: include the multiplexer role explicitly
multiplexer_tmux_enabled: true
multiplexer_zellij_enabled: true
multiplexer_screen_enabled: false
```

Add `marcstraube.common.multiplexer` to the playbook role list (or
keep using `base_system.yml`, which already includes it).

### `security.yml` and `backup.yml` task files merged into `base_system.yml` (#96)

`playbooks/tasks/security.yml` and `playbooks/tasks/backup.yml` are
removed. All security roles (and `restic`, with the `never` tag) are
integrated directly into `playbooks/tasks/base_system.yml`. Firewalld
now runs before Docker so its rules are not clobbered by unmanaged
nftables.

The `security` tag is preserved on all security roles, so
`--tags security` filtering keeps working.

#### Before

```yaml
# Old: separate include paths for security and backup
- import_playbook: marcstraube.common.base_system
  tags: [base-system]

- import_tasks: tasks/security.yml
  tags: [security]

- import_tasks: tasks/backup.yml
  tags: [backup, never]
```

#### After

```yaml
# v2.0.0: all baked into base_system.yml
- import_playbook: marcstraube.common.base_system
  tags: [base-system]
# tasks/security.yml and tasks/backup.yml no longer exist;
# --tags security and --tags backup still work against base_system
```

Run `ansible-playbook ... --tags security` (or `--tags backup`) for
the filtered behavior. Backup remains tagged `never` and must be
invoked explicitly.

### `wireguard` — NM-managed mode for clients (#102)

A new `mode: nm` is added to entries of `wireguard_interfaces`. It
deploys a `.nmconnection` keyfile and lets NetworkManager handle
routing, DNS, and lifecycle, instead of wg-quick + systemd. This is
the recommended mode for laptop clients with full-tunnel
`AllowedIPs`, where wg-quick previously broke host connectivity.

The VPN autoconnect dispatcher in `networkmanager` is also changed to
use `nmcli connection up/down` (instead of `systemctl start/stop
wg-quick@…`). Clients must use `mode: nm` for the dispatcher to manage
the tunnel correctly.

#### Before

```yaml
# v1.x / pre-#102: wg-quick-managed client
wireguard_interfaces:
  - name: wg0
    mode: client
    address: '10.100.0.2/32'
    private_key: '...'
    dns: ['10.100.0.1']
    peers:
      - name: 'home-server'
        public_key: '...'
        endpoint: 'vpn.example.com:51820'
        allowed_ips: '0.0.0.0/0'
        persistent_keepalive: 25
```

#### After

```yaml
# v2.0.0: NetworkManager-managed client (recommended for laptops)
wireguard_interfaces:
  - name: wg0
    mode: nm                  # new: NM keyfile instead of wg-quick
    address: '10.100.0.2/32'
    private_key: '...'
    dns: ['10.100.0.1']
    dns_search: 'example.lan'
    dns_priority: -10         # negative => use these DNS first
    never_default: false      # true to avoid setting default route
    nm_autoconnect: false     # let dispatcher start/stop the tunnel
    peers:
      - name: 'home-server'
        public_key: '...'
        endpoint: 'vpn.example.com:51820'
        allowed_ips: '0.0.0.0/0'
        persistent_keepalive: 25
```

Server interfaces should keep `mode: server`. Headless clients without
NetworkManager can keep `mode: client` (wg-quick).

### `networkmanager` — `toggle_wifi` removed (#114)

The per-connection `toggle_wifi: true` property and its dispatcher
script `10-wifi-auto-toggle.sh` are removed. With `rp_filter=2`
(systemd default since 2018, also the default on the project's laptop
inventories), parallel ETH + WiFi on the same LAN works without the
auto-toggle, and the dispatcher caused rfkill-state persistence bugs
that warranted its removal.

`toggle_wifi: true` is silently ignored from v2.0.0 on. Inventories
that relied on it lose the auto-WiFi-off behavior — WiFi will stay on
alongside ethernet. See [#113](https://github.com/marcstraube/ansible-collection-common/issues/113)
for the full root-cause analysis.

#### Before

```yaml
networkmanager_connections:
  ethernet:
    "Office":
      ifname: "eth0"
      toggle_wifi: true   # auto-turn-off WiFi when this connection is up
```

#### After

```yaml
networkmanager_connections:
  ethernet:
    "Office":
      ifname: "eth0"
      # toggle_wifi removed — drop the field, no replacement
```

If WiFi-off-on-ethernet is still desired, manage it externally
(e.g. via a custom dispatcher or `nmcli radio wifi off` on demand).

### `firejail` — disabled by default with cleanup path (#118)

`firejail_enabled` now defaults to `false`. On modern Wayland +
AppArmor + xdg-desktop-portal stacks the firecfg auto-wrap breaks
portal openuri calls (Electron apps, native-messaging hosts like
KeePassXC-Browser) more than it adds defense-in-depth — AppArmor and
in-browser sandboxes cover the kernel-level confinement firejail was
historically valued for.

When the role is invoked with `firejail_enabled: false`, it now runs
a cleanup path that removes firecfg symlinks, the pacman hook,
`globals.local`, role-managed `*.local` profile overrides, and (by
default) the `firejail` package itself. The cleanup is reachable
unconditionally — `base_system.yml` no longer wraps the role in a
`when: firejail_enabled` guard, the role dispatches install / configure
or cleanup internally.

A new toggle `firejail_remove_package: true` (default) controls whether
the cleanup also uninstalls the package. Set to `false` to keep the
package on disk for ad-hoc CLI sandboxing while disabling the role's
auto-wrap.

#### Before

```yaml
# v1.x: firejail on by default
firejail_enabled: true  # default
```

#### After

```yaml
# v2.0.0: firejail off by default
# On the next run, the cleanup path removes firecfg symlinks,
# pacman hook, globals.local, role-managed *.local overrides,
# and (by default) the firejail package.

# To restore previous behavior:
firejail_enabled: true

# To disable firejail but keep the package for ad-hoc use:
firejail_enabled: false
firejail_remove_package: false
```

### `shell` — role moved from `marcstraube.desktop` to `marcstraube.common` (#137)

The `shell` role (zsh, bash, fish, Oh My Zsh, Starship prompt) was
moved verbatim from `marcstraube.desktop` to `marcstraube.common`. All
inventory variables (`shell_*`) keep their names; only the FQCN
changes. This is covered by the v2.0.0 major bump in both collections.

#### Before

```yaml
# v1.x: role lived in the desktop collection
- hosts: workstations
  roles:
    - marcstraube.desktop.shell
```

#### After

```yaml
# v2.0.0: role lives in the common collection
- hosts: all
  roles:
    - marcstraube.common.shell
```

`base_system.yml` already includes the role in v2.0.0, so most
consumers only need to remove a direct desktop-collection reference if
they had one.

### `networkmanager` — generic `settings:` dict (#153)

Three per-connection fields are removed and must be migrated into the
new generic `settings:` dict, which takes nmcli-native keys verbatim.

#### Field renames

| Old per-connection field         | New `settings:` key                                |
|----------------------------------|----------------------------------------------------|
| `mac: "..."`                     | `802-11-wireless.cloned-mac-address: "..."` (WiFi) |
| `dhcp_send_hostname_ipv4: true`  | `ipv4.dhcp-send-hostname: "yes"`                   |
| `dhcp_send_hostname_ipv4: false` | `ipv4.dhcp-send-hostname: "no"`                    |
| `dhcp_send_hostname_ipv6: true`  | `ipv6.dhcp-send-hostname: "yes"`                   |
| `dhcp_send_hostname_ipv6: false` | `ipv6.dhcp-send-hostname: "no"`                    |

Note: the old `mac:` field was overloaded — it was passed to the
`community.general.nmcli` module as the device MAC selector *and* used
for the cloned MAC address. The new schema separates these:

- `802-11-wireless.cloned-mac-address` — clone / randomize on connect
- `802-11-wireless.mac-address` — device selector (lock connection to this MAC)
- `802-3-ethernet.mac-address` — ethernet device selector

#### Before

```yaml
networkmanager_connections:
  ethernet:
    "Office":
      ifname: "eth0"
      dhcp_send_hostname_ipv4: false
      dhcp_send_hostname_ipv6: false
  wifi:
    "Home WiFi":
      ssid: "MyNetwork"
      mac: "random"
```

#### After

```yaml
networkmanager_connections:
  ethernet:
    "Office":
      ifname: "eth0"
      settings:
        ipv4.dhcp-send-hostname: "no"
        ipv6.dhcp-send-hostname: "no"
  wifi:
    "Home WiFi":
      ssid: "MyNetwork"
      settings:
        802-11-wireless.cloned-mac-address: "random"
```

#### Value formats

For ternary-boolean keys (`*.dhcp-send-hostname`, `connection.metered`,
etc.) any of nmcli's accepted forms work: `yes`/`true`/`1`,
`no`/`false`/`0`, or `default`/`unknown`/`-1`. The role normalizes both
sides before comparing, so the choice is purely cosmetic.

For all other keys (strings, MAC addresses, enums like `random`), use
the value verbatim — comparison is case-sensitive against the exact
string nmcli returns.

The global daemon-config defaults (`networkmanager_dhcp_send_hostname_ipv4`
/ `_ipv6`) are unaffected by this change — they continue to write the
`[connection]` section of `/etc/NetworkManager/conf.d/00-ansible.conf`.
