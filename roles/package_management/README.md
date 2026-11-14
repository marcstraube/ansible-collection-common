# marcstraube.common.package_management

Configure package managers, repositories, and related tools across multiple OS families.

## Description

Manages package manager configuration, third-party repositories, and related tools across
Arch Linux, Debian, and RHEL/Rocky. Includes pacman/reflector/paru (Arch),
apt/unattended-upgrades (Debian), and dnf/EPEL/RPM Fusion (RHEL/Rocky) with dnf5
support for Rocky 10.

## Requirements

- ansible-core >= 2.19
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

`pacman.conf` includes every `*.conf` file in `/etc/pacman.d/ignore.d`, so
other roles can add `IgnorePkg` and `IgnoreGroup` lines in files of their
own. pacman merges them with `pacman_ignore_packages` and
`pacman_ignore_groups`. The role places `00-package-management.conf` in the
directory, because pacman fails on an `Include` glob that matches no file.
An ignored package is still installed by an explicit `pacman -S` or
`pacman -U`.

### Arch Linux - Dependency Packages

| Variable                     | Default | Description                                   |
|------------------------------|---------|-----------------------------------------------|
| `pacman_dependency_packages` | `[]`    | Repository packages installed as dependencies |

When a package depends on a virtual package with several providers (for
example `tessdata` for `tesseract`, or `cron` for a package that needs a
cron daemon), an unattended install (`pacman --noconfirm`, paru, the
`community.general.pacman` module) takes the first provider, not the one you
want. Listing the wanted provider in a role's package list installs it
explicitly, so it stays when nothing needs it anymore and does not show up as
a dependency.

The role installs every missing package in `pacman_dependency_packages`
(`--needed`) as dependency right after `pacman.conf`, the repository keys and the
package cache are in place, and before the AUR helper or any later role
installs packages. A later install that needs the virtual package then finds
the listed provider and keeps it. A listed package that is already installed
explicitly is marked as dependency (`pacman -D --asdeps`); the role reads the
install reason first, so a second run changes nothing. The list takes
packages from the configured repositories only, not from the AUR. Debian and
RHEL/Rocky ignore it.

```yaml
pacman_dependency_packages:
  - 'tesseract-data-deu'  # instead of tesseract-data-afr for tesseract
  - 'fcron'               # instead of cronie for packages that need cron
```

A package installed as dependency is an orphan as soon as no installed
package requires it (`pacman -Qtdq`): before the consumer is installed, and
after it is removed. `marcstraube.archlinux.system_update` reports orphans
(`system_update_warn_orphans`) and removes them with
`system_update_auto_remove_orphans: true`. That role runs before this one in
the site playbook, so with the removal enabled a listed provider that nothing
needs is removed and installed again on every run. List only providers that a
package on the host requires.

The list chooses a provider only while none is installed. A listed provider
that conflicts with an installed one (`fcron` while `cronie` is installed)
fails the run, since `--noconfirm` declines removing the installed package;
`--check` does not detect the conflict. Replace the installed provider by hand
first, e.g. `pacman -S fcron` and confirm the removal of `cronie`.

The list is the one place that enforces a provider for the whole host. A
role that needs a virtual package and brings a default provider installs it
only while the virtual package is not satisfied yet (`pacman -T cron` exits
127) and leaves an installed provider alone. Since this role runs first, a
provider in the list takes precedence over every role default, and the roles
never pull a second, conflicting provider.

### Arch Linux - Reflector

| Variable                  | Default | Description                                                   |
|---------------------------|---------|---------------------------------------------------------------|
| `reflector_enabled`       | `true`  | Enable reflector mirror management                            |
| `reflector`               | dict    | Mirror config (countries, protocol, latest, sort, age, score) |
| `reflector_timer_enabled` | `true`  | Enable periodic mirror updates                                |

### Arch Linux - Makepkg

| Variable                    | Default      | Description                                     |
|-----------------------------|--------------|-------------------------------------------------|
| `makepkg_packager`          | `''`         | Packager name/email for built packages          |
| `makepkg_cflags`            | `''`         | Custom C compiler flags (empty = Arch defaults) |
| `makepkg_ltoflags`          | `-flto=auto` | LTO flags                                       |
| `makepkg_ccache_enabled`    | `false`      | Enable ccache                                   |
| `makepkg_sccache_enabled`   | `false`      | Enable sccache (Rust)                           |
| `makepkg_distcc_enabled`    | `false`      | Enable distcc                                   |
| `makepkg_devtools_enabled`  | `false`      | Install devtools (chrooted builds)              |
| `makepkg_nvchecker_enabled` | `false`      | Install nvchecker (upstream version tracking)   |
| `makepkg_check`             | `true`       | Run check() in PKGBUILDs                        |
| `makepkg_tmpdir`            | `''`         | TMPDIR for builds (empty = /tmp; see below)     |
| `makepkg_sign`              | `false`      | Sign built packages                             |
| `makepkg_debug`             | `true`       | Build with debug symbols                        |
| `makepkg_lto`               | `true`       | Build with LTO                                  |

`makepkg_tmpdir` is exported as `TMPDIR` in `/etc/makepkg.conf`. Set it when
`/tmp` is mounted `noexec`: some PKGBUILDs run pip or meson, which execute
files they create below `TMPDIR`, and the build fails with `Permission denied`.
The role creates the directory with mode `1777` and a tmpfiles.d entry that
recreates it at boot and removes files older than 10 days. The directory must
be on a filesystem without `noexec`, for example `/var/cache/makepkg`.

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

### Arch Linux - Paru PKGBUILD Repositories

| Variable                     | Default | Description                    |
|------------------------------|---------|--------------------------------|
| `paru_pkgbuild_repositories` | `[]`    | PKGBUILD repositories for paru |

paru (2.0 and later) builds packages from repositories of PKGBUILDs besides
the AUR. Each entry becomes a `[name]` section in `/etc/paru.conf` and needs
`name` and exactly one of `url` and `path`:

| Key                | paru.conf         | Description                                                    |
|--------------------|-------------------|----------------------------------------------------------------|
| `name`             | `[name]`          | Unique name: letters, digits, `_`, `+`, `-`                    |
| `url`              | `Url`             | Git URL: `https://`, `ssh://` or `file:///`                    |
| `path`             | `Path`            | Absolute path of an existing directory on the host             |
| `subdirectory`     | `Path`            | With `url`: search for PKGBUILDs below this relative directory |
| `depth`            | `Depth`           | Directory levels scanned, `1` is the top only (paru: `2`)      |
| `skip_review`      | `SkipReview`      | With `url`: skip the review in manual runs (`false`)           |
| `generate_srcinfo` | `GenerateSrcinfo` | Regenerate `.SRCINFO` even if present (`false`)                |

```yaml
paru_pkgbuild_repositories:
  - name: 'mypkgbuilds'
    url: 'https://git.example.com/johndoe/pkgbuilds.git'
    subdirectory: 'packages'
  - name: 'local'
    path: '/srv/pkgbuilds'
```

The role fails before any change when an entry is invalid, when a `path`
directory is not safe (see below) or when the list is set and `aur_helper` is
not `paru`: yay has no PKGBUILD repositories. Every value becomes a line in
`paru.conf`, so `path` and `subdirectory` must not contain whitespace or
control characters, and `subdirectory` must not start with `/` or contain `..`.

`pacman_custom_repositories` adds binary repositories that pacman itself
installs from, signed packages built elsewhere. A PKGBUILD repository holds
build recipes instead: paru builds its packages on the host as `aur_builder`,
like AUR packages, and pacman never sees the repository.

**A PKGBUILD repository shadows the AUR.** paru prefers its packages over AUR
packages of the same name, and a PKGBUILD runs arbitrary code during the build.
Only add repositories from a trusted source. A `path` repository has no review
step at all. For a `url` repository, `skip_review: false` only shows changes
in manual paru runs, and only the commits paru has not marked as seen yet.
paru marks the downloaded commits as seen even with `--noconfirm`, which skips
the review: every download the role starts and every install by another role
(`kewlfft.aur.aur`) does that, so a later manual `paru -Syu` does not show
these changes. When the source is not under the administrator's control,
review new commits in the clone yourself (`git log -p` in
`<paru_clone_dir>/repo/<name>`) before installing from it with Ansible.

A `url` repository is cloned to `<paru_clone_dir>/repo/<name>`. The role
downloads it as `aur_builder` with `paru -Sy --pkgbuilds` when the section of
a `url` repository in `paru.conf` is added or changed or a clone is missing,
so other roles can install its packages in the same run; this does not
refresh the pacman databases. Other changes to `paru.conf` do not download,
so a run without access to the Git server only fails when a `url` repository
changed. Later changes arrive with every manual `paru -Sy` or `paru -Syu`.
When paru is not installed yet, for example with
`--tags package-management:configure` on a new host, the role skips the
download and says so; the next run that finds paru catches up.

Only `https://`, `ssh://` and `file:///` URLs are accepted: `http://` and
`git://` have no transport protection, and the downloaded build recipes are
installed as root. A `url` must not contain credentials: neither
`user:password@` nor, in an `https://` URL, a token or user name before the
`@` (`https://<token>@host/...`). `paru.conf` is world-readable and the clone
stores the URL in `.git/config`. An `ssh://` URL may name the login user
(`ssh://git@host/...`). For a private repository use `ssh://` with an
SSH key of `aur_builder`, or keep `https://` and give `aur_builder` a Git
credential helper that reads the token from a file only it can read. The role sets up neither the key nor the host key:
put the key below `<aur_builder_home>/.ssh/` and the server's host key into
`<aur_builder_home>/.ssh/known_hosts`, both owned by `aur_builder`. Without a
known host key the download fails, since it runs without a terminal.

paru itself keeps fetching from the URL a clone was made with, even after the
`Url` in `paru.conf` changed. The role therefore compares the origin of each
clone with its `url`, removes the clone of a repository whose `url` changed and
downloads it again from the new URL. Installed packages are rebuilt from the
new source on their next upgrade, once it offers a newer version. Clones of
repositories removed from the list stay in place. paru normalizes a `Url`
before cloning (WHATWG URL rules), so the role only accepts a `url` in a form
paru keeps: printable ASCII without whitespace and without the characters
paru percent-encodes (`"`, `<`, `>`, `\`, `^`, `` ` ``, `{`, `|`, `}`) or a
query or fragment (`?`, `#`), lowercase scheme and host, no `:443` with
`https://`, a path of at least `/` after an `https://` host, `file:///`
instead of `file://localhost/` and no `.` or `..` path segments (also not as
`%2e`). A `url` that paru still rewrites, such as an IPv4 host in hexadecimal
notation, is cloned again on every run.

paru reads a `path` repository in place on every run, so no download is
needed. paru builds its packages inside that directory and afterwards cleans
the package directory with `git restore` and `git clean -fx`
(`kewlfft.aur.aur` always passes `--cleanafter`). The directory must
therefore be a Git work tree that `aur_builder` can write to; uncommitted
changes and untracked files in a package directory are discarded after its
build. Outside a Git work tree the build installs the package but paru exits
with an error.

Whoever can write to a `path` directory can slip in a package that is
installed as root. The role therefore fails when the directory does not exist,
is a symlink or another non-directory, grants write permission to its group or
others, or is not owned by root or `aur_builder_user`. The role creates
`aur_builder` itself: on a new host the directory can belong to root until
the first run, and is then handed to `aur_builder` (`chown -R`) for builds.

Packages from these repositories install like AUR packages, for example with
`kewlfft.aur.aur` as `aur_builder`. `aur_only: true` passes `--aur`, which
excludes PKGBUILD repositories.

### Arch Linux - Shared AUR Build Infrastructure

These apply to whichever helper `aur_helper` selects.

| Variable           | Default                | Description                       |
|--------------------|------------------------|-----------------------------------|
| `aur_shared_group` | `aur_shared`           | Group owning the clone directory  |
| `aur_shared_users` | `[]`                   | Users sharing the AUR clone cache |
| `yay_clone_dir`    | `/var/cache/yay/clone` | Shared clone directory for yay    |

`paru_clone_dir` and `yay_clone_dir` stay separate: the two helpers track VCS
state differently, so they must not build in the same tree. Everything else —
group, ownership, permissions, git `safe.directory` — is shared between them.

Unlike paru, yay has no system-wide configuration file. It reads
`~/.config/yay/config.json`, so the role writes `buildDir` there for the
builder user and for every `aur_shared_users` entry, merging into an existing
file rather than replacing it. The wrapper additionally passes `--builddir`,
so a manual call lands in the shared tree regardless of the caller's own
config.

The shared clone directory is owned by the `aur_builder` user and group-owned
by `aur_shared_group` with the SGID bit. All builds run as `aur_builder` under an
enforced `022` umask (set through the user's GECOS `umask=` field and a
sudoers `umask_override`), so cloned sources and built packages stay readable
and executable for the shared group even when a restrictive default
umask (such as the `hardening` role's `0077`) is in effect. ACLs are
deliberately **not** used: default ACLs make `libarchive`/`bsdtar` skip its
final `chmod` during source extraction, which would bake the restrictive
modes of the initial secure `open()` into the built packages. Because
`makepkg` only chmods paths for their owner, the tree is owned by a single
user; other group members get read/execute access (never write)
through the SGID bit and the enforced umask.

Manual upgrades must therefore run as the build user. The role installs a
`/usr/local/bin/paru` or `/usr/local/bin/yay` wrapper that transparently
re-executes the helper as `aur_builder` via `sudo`, so `paru -Syu` or
`yay -Syu` works from any `wheel` account (authenticating with the caller's
own password) or as `root`. Never invoke the real `/usr/bin/paru` or
`/usr/bin/yay` directly as a login user — that would create clone entries
owned by the wrong user and lock the builder out on the next run.

All sudoers rules for the builder (passwordless `pacman`, the enforced umask,
and the wrapper rule for `wheel`) live in
`/etc/sudoers.d/99-role-package_management`. The `NN-role-<role>` name keeps
the file in place when the `sudo` role purges unmanaged drop-ins.

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

| Variable                        | Default | Description                                         |
|---------------------------------|---------|-----------------------------------------------------|
| `apt_backports_enabled`         | `false` | Enable Debian Backports                             |
| `apt_non_free_enabled`          | `false` | Add `non-free` to Debian deb822 Components          |
| `apt_non_free_firmware_enabled` | `false` | Add `non-free-firmware` to Debian deb822 Components |
| `apt_custom_repos`              | `[]`    | Custom repository definitions (deb822 fields)       |

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
| `dnf_remi_php_version`            | `''`    | Remi PHP module stream (EL9; `''` = none)   |
| `dnf_rpmfusion_free_enabled`      | `false` | Enable RPM Fusion Free                      |
| `dnf_rpmfusion_nonfree_enabled`   | `false` | Enable RPM Fusion Nonfree                   |
| `dnf_elrepo_enabled`              | `false` | Enable ELRepo                               |
| `dnf_copr_repos`                  | `[]`    | COPR repositories                           |
| `dnf_custom_repos`                | `[]`    | Custom repository definitions               |

## Tags

| Tag                               | Scope                           |
|-----------------------------------|---------------------------------|
| `package-management`              | All tasks                       |
| `package-management:install`      | AUR helper installation         |
| `package-management:configure`    | Package manager configuration   |
| `package-management:dependencies` | Dependency package installation |
| `package-management:repos`        | Repository management           |
| `package-management:tools`        | Additional tool installation    |

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

## References

- [Pacman](https://wiki.archlinux.org/title/Pacman) — Arch Linux package manager
- [Makepkg](https://wiki.archlinux.org/title/Makepkg) — Arch Linux package build utility
- [APT Configuration](https://wiki.debian.org/AptConfiguration) — Debian package manager configuration
- [DNF Documentation](https://dnf.readthedocs.io/) — Red Hat package manager

## License

MIT

## Author

Marc Straube
