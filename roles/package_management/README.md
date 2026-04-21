# marcstraube.common.package_management

Configure package managers, repositories, and related tools across multiple OS families.

## Description

Manages package manager configuration, third-party repositories, and related tools across
Arch Linux, Debian, and RHEL/Rocky. Includes pacman/reflector/paru (Arch),
apt/unattended-upgrades (Debian), and dnf/EPEL/RPM Fusion (RHEL/Rocky) with dnf5
support for Rocky 10.

## Requirements

- `ansible-core >= 2.17`
- Collections: `community.general`, `kewlfft.aur` (Arch only)

## Supported Platforms

| Platform                  | Notes                                                            |
|---------------------------|------------------------------------------------------------------|
| Arch Linux                | pacman, reflector, makepkg, paru (AUR helper), paccache, pkgfile |
| Debian Trixie             | apt, apt-file, needrestart, unattended-upgrades, debsums         |
| EL 9 (Rocky, Alma, RHEL)  | dnf4, EPEL, Remi, RPM Fusion, ELRepo, dnf-automatic              |
| EL 10 (Rocky, Alma, RHEL) | dnf5 (modularity removed, packages gated per version)            |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Global

| Variable                     | Default | Description                |
|------------------------------|---------|----------------------------|
| `package_management_enabled` | `true`  | Enable/disable entire role |

### Arch Linux - Pacman

| Variable                       | Default                     | Description                           |
|--------------------------------|-----------------------------|---------------------------------------|
| `pacman_parallel_downloads`    | `5`                         | Number of parallel downloads (1-10)   |
| `pacman_color`                 | `true`                      | Enable color output                   |
| `pacman_verbose_pkg_lists`     | `true`                      | Verbose package lists during upgrades |
| `pacman_check_space`           | `true`                      | Check available disk space            |
| `pacman_ilovecandy`            | `true`                      | Animated pacman easter egg            |
| `pacman_sig_level`             | `Required DatabaseOptional` | Package signature verification        |
| `pacman_hold_packages`         | `[pacman, glibc]`           | Packages that won't be removed        |
| `pacman_ignore_packages`       | `[]`                        | Packages that won't be upgraded       |
| `pacman_testing_repos_enabled` | `false`                     | Enable testing repositories           |
| `pacman_multilib_enabled`      | `false`                     | Enable 32-bit multilib support        |
| `pacman_custom_repositories`   | `[]`                        | Custom repository definitions         |
| `pacman_download_user`         | `alpm`                      | Sandboxed download user               |
| `pacman_hook_dir`              | `/etc/pacman.d/hooks`       | Custom hook directory                 |

### Arch Linux - Reflector

| Variable                  | Default | Description                                                   |
|---------------------------|---------|---------------------------------------------------------------|
| `reflector_enabled`       | `true`  | Enable reflector mirror management                            |
| `reflector`               | dict    | Mirror config (countries, protocol, latest, sort, age, score) |
| `reflector_timer_enabled` | `true`  | Enable periodic mirror updates                                |

### Arch Linux - Makepkg

| Variable                 | Default      | Description                                     |
|--------------------------|--------------|-------------------------------------------------|
| `makepkg_packager`       | `''`         | Packager name/email for built packages          |
| `makepkg_cflags`         | `''`         | Custom C compiler flags (empty = Arch defaults) |
| `makepkg_ltoflags`       | `-flto=auto` | LTO flags                                       |
| `makepkg_ccache_enabled` | `false`      | Enable ccache                                   |
| `makepkg_distcc_enabled` | `false`      | Enable distcc                                   |
| `makepkg_check`          | `true`       | Run check() in PKGBUILDs                        |
| `makepkg_sign`           | `false`      | Sign built packages                             |
| `makepkg_debug`          | `true`       | Build with debug symbols                        |
| `makepkg_lto`            | `true`       | Build with LTO                                  |

### Arch Linux - Paru (AUR Helper)

| Variable                | Default                 | Description                      |
|-------------------------|-------------------------|----------------------------------|
| `aur_helper`            | `paru`                  | AUR helper (paru, yay, or empty) |
| `aur_builder_user`      | `aur_builder`           | System user for AUR builds       |
| `paru_clone_dir`        | `/var/cache/paru/clone` | Shared clone directory           |
| `paru_clean_after`      | `true`                  | Remove untracked build files     |
| `paru_remove_make`      | `true`                  | Remove makedepends after build   |
| `paru_bottom_up`        | `true`                  | Show newest results at bottom    |
| `paru_sudo_loop`        | `false`                 | Keep sudo alive during builds    |
| `paru_devel`            | `false`                 | Check VCS packages for updates   |
| `paru_news_on_upgrade`  | `false`                 | Show Arch news on upgrade        |
| `paru_combined_upgrade` | `true`                  | Combined repo+AUR upgrade menu   |
| `paru_batch_install`    | `true`                  | Queue builds, install together   |
| `paru_pgp_fetch`        | `true`                  | Auto-import PGP keys             |
| `paru_skip_review`      | `false`                 | Skip PKGBUILD review             |
| `paru_fail_fast`        | `true`                  | Exit on first build failure      |
| `paru_upgrade_menu`     | `true`                  | Detailed upgrade menu            |
| `paru_provides`         | `true`                  | Search for provider packages     |
| `paru_sort_by`          | `votes`                 | Sort search results              |
| `paru_search_by`        | `name-desc`             | Search by field                  |
| `paru_shared_users`     | `[]`                    | Users sharing AUR clone cache    |

### Arch Linux - Tools

| Variable                          | Default | Description                          |
|-----------------------------------|---------|--------------------------------------|
| `paccache_enabled`                | `true`  | Enable paccache timer                |
| `paccache_args`                   | `-rk2`  | Paccache arguments                   |
| `pacman_rebuild_detector_enabled` | `true`  | Install rebuild-detector             |
| `pacman_pkgfile_enabled`          | `true`  | Install pkgfile (command-not-found)  |
| `pacman_informant_enabled`        | `false` | Install informant (AUR news blocker) |

### Debian/Ubuntu - APT

| Variable                          | Default | Description                  |
|-----------------------------------|---------|------------------------------|
| `apt_install_recommends`          | `true`  | Install recommended packages |
| `apt_install_suggests`            | `false` | Install suggested packages   |
| `apt_assume_yes`                  | `true`  | Non-interactive mode         |
| `apt_allow_downgrades`            | `true`  | Allow package downgrades     |
| `apt_languages`                   | `none`  | Translation downloads        |
| `apt_periodic_enabled`            | `true`  | Enable periodic updates      |
| `apt_periodic_update`             | `1`     | Update interval (days)       |
| `apt_periodic_autoclean`          | `7`     | Auto-clean interval (days)   |
| `apt_preferences`                 | `[]`    | Package pinning rules        |
| `apt_unattended_upgrades_enabled` | `false` | Enable unattended-upgrades   |
| `apt_apt_file_enabled`            | `true`  | Install apt-file             |
| `apt_needrestart_enabled`         | `true`  | Install needrestart          |
| `apt_debsums_enabled`             | `false` | Install debsums              |

### Debian/Ubuntu - Repositories

| Variable                          | Default | Description                           |
|-----------------------------------|---------|---------------------------------------|
| `apt_backports_enabled`           | `false` | Enable Debian Backports               |
| `apt_custom_repos`                | `[]`    | Custom repository definitions         |

### RHEL/Rocky - DNF

| Variable                           | Default | Description                   |
|------------------------------------|---------|-------------------------------|
| `dnf_fastestmirror`                | `true`  | Enable fastest mirror         |
| `dnf_max_parallel_downloads`       | `10`    | Maximum parallel downloads    |
| `dnf_best`                         | `true`  | Install only best versions    |
| `dnf_install_weak_deps`            | `true`  | Install weak dependencies     |
| `dnf_keepcache`                    | `false` | Keep downloaded packages      |
| `dnf_clean_requirements_on_remove` | `true`  | Auto-remove unneeded deps     |
| `dnf_proxy`                        | `''`    | HTTP proxy for downloads      |
| `dnf_automatic_enabled`            | `false` | Enable dnf-automatic          |
| `dnf_utils_enabled`                | `true`  | Install dnf-utils (dnf4 only) |
| `dnf_needs_restarting_enabled`     | `true`  | Install yum-utils (dnf4 only) |

### RHEL/Rocky - Repositories

| Variable                          | Default | Description                                 |
|-----------------------------------|---------|---------------------------------------------|
| `dnf_epel_enabled`                | `true`  | Enable EPEL                                 |
| `dnf_powertools_enabled`          | `false` | Enable CRB/PowerTools                       |
| `dnf_remi_enabled`                | `false` | Enable Remi (PHP, MariaDB)                  |
| `dnf_rpmfusion_free_enabled`      | `false` | Enable RPM Fusion Free                      |
| `dnf_rpmfusion_nonfree_enabled`   | `false` | Enable RPM Fusion Nonfree                   |
| `dnf_elrepo_enabled`              | `false` | Enable ELRepo                               |
| `dnf_copr_repos`                  | `[]`    | COPR repositories                           |
| `dnf_custom_repos`                | `[]`    | Custom repository definitions               |

## Tags

| Tag                            | Scope                         |
|--------------------------------|-------------------------------|
| `package-management`           | All tasks                     |
| `package-management:install`   | AUR helper installation       |
| `package-management:configure` | Package manager configuration |
| `package-management:repos`     | Repository management         |
| `package-management:tools`     | Additional tool installation  |

## Example Playbook

```yaml
- name: Configure package management
  hosts: all
  become: true
  tasks:
    - name: Include package_management role
      ansible.builtin.include_role:
        name: marcstraube.common.package_management
      tags:
        - package-management
```

## Testing

```bash
cd roles/package_management
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

### Rocky 10 / dnf5

- `dnf-plugins-core`, `dnf-utils`, `yum-utils` are automatically skipped (removed in dnf5)
- `fastestmirror` is built into dnf5 (no plugin needed, config option still works)

## License

MIT

## Author

Marc Straube
