# marcstraube.common.php

Install and manage multiple PHP versions side by side with per-version FPM
and extensions, plus optional Composer support.

## Description

This role manages PHP installation across multiple platforms, supporting
parallel installation of multiple PHP versions. Each version can have its own
set of extensions and FPM configuration. On Arch Linux, the role detects the
current repo version and uses AUR packages for non-current versions.

Configuration is managed via drop-in php.ini overrides (upstream untouched)
and full FPM pool templates.

## Requirements

- ansible-core >= 2.17
- Collections: `community.general`, `kewlfft.aur` (Arch only)
- **Debian/Ubuntu**: Sury PHP repository is set up automatically by this role
- **RHEL/Rocky**: Remi + EPEL repository via `package_management`
  (`dnf_remi_enabled: true`, `dnf_epel_enabled: true`)

## Supported Platforms

| Platform                   | Notes                                                          |
|----------------------------|----------------------------------------------------------------|
| Arch Linux                 | Official repo for current PHP, AUR for legacy versions         |
| Debian Trixie              | Sury repository (multi-version side-by-side)                   |
| EL 9 (Rocky, Alma, RHEL)   | AppStream (default, single stream) or Remi (`php_redhat_repo`) |
| EL 10 (Rocky, Alma, RHEL)  | AppStream (default, single stream) or Remi (`php_redhat_repo`) |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable      | Default | Description         |
|---------------|---------|---------------------|
| `php_enabled` | `true`  | Enable the php role |

### PHP-FPM

| Variable          | Default | Description                       |
|-------------------|---------|-----------------------------------|
| `php_fpm_enabled` | `true`  | Enable and start PHP-FPM services |

### PHP Versions

| Variable              | Default                 | Description                                       |
|-----------------------|-------------------------|---------------------------------------------------|
| `php_versions`        | `[{version: 'auto'}]`   | PHP versions to install (list of dicts)           |
| `php_default_version` | `'auto'`                | Default PHP version for CLI                       |
| `php_redhat_repo`     | `'appstream'`           | RHEL repo: `appstream` (single) or `remi` (multi) |

#### `php_redhat_repo` — AppStream vs. Remi

| Aspect                 | `'appstream'` (default)                   | `'remi'`                                      |
|------------------------|-------------------------------------------|-----------------------------------------------|
| Source                 | Distro AppStream (`dnf module` streams)   | Remi Collet's third-party repo                |
| Package names          | `php-cli`, `php-fpm`, `php-<ext>`         | `php<XX>-php-cli`, `php<XX>-php-<ext>`        |
| Paths                  | `/etc/php.d`, `/etc/php-fpm.d`            | `/etc/opt/remi/php<XX>/...`                   |
| Available versions     | 8.0, 8.1, 8.2 on Rocky 9 (one at a time)  | 5.6 through current stable, side-by-side      |
| Update cadence         | RHEL lifecycle (conservative)             | Upstream release day                          |
| Lifecycle              | Application-stream support window         | Upstream PHP support per minor                |
| `php_versions` length  | Must be 1 (streams are exclusive)         | Multiple versions allowed                     |

When `php_redhat_repo: 'appstream'` and `php_versions[0].version` differs
from the active module stream, the role fails with a hint to either
switch the stream (`sudo dnf module reset -y php && sudo dnf module enable
-y php:<X.Y>`) or use Remi.

When `php_redhat_repo: 'remi'`, the Remi repository must already be on
the host. Remi is multi-purpose (PHP, MariaDB, Redis, …) and is managed
by `marcstraube.common.package_management`, not by this role. Enable it
in inventory:

```yaml
dnf_remi_enabled: true
# Optional: pin the active PHP module stream (Remi's modular variant)
dnf_remi_php_version: '8.2'
```

Then run the `package_management` role before this one. The PHP role
asserts the Remi repo is present and fails fast with this hint
otherwise.

Each entry in `php_versions` supports:

| Key            | Default | Description                                                          |
|----------------|---------|----------------------------------------------------------------------|
| `version`      | —       | PHP version (e.g., `'8.3'`, `'8.4'`)                                 |
| `extensions`   | `[]`    | Extensions to install (e.g., `curl`)                                 |
| `fpm`          | `true`  | Install and enable PHP-FPM                                           |
| `ini`          | `{}`    | Per-version php.ini overrides (keys without `php_ini_` prefix)       |
| `fpm_pool`     | `{}`    | Per-version FPM pool overrides (keys without `php_fpm_pool_` prefix) |

### PHP Configuration (php.ini)

Deployed as drop-in override to `conf.d/99-ansible.ini`. Upstream php.ini stays
untouched — only managed settings are overridden.

| Variable                                 | Default                                | Description                 |
|------------------------------------------|----------------------------------------|-----------------------------|
| `php_ini_memory_limit`                   | `'128M'`                               | Memory limit per script     |
| `php_ini_max_execution_time`             | `30`                                   | Max execution time (sec)    |
| `php_ini_max_input_time`                 | `60`                                   | Max input parsing time      |
| `php_ini_max_input_vars`                 | `1000`                                 | Max input variables         |
| `php_ini_file_uploads`                   | `'On'`                                 | Allow file uploads          |
| `php_ini_upload_max_filesize`            | `'2M'`                                 | Max upload file size        |
| `php_ini_post_max_size`                  | `'8M'`                                 | Max POST data size          |
| `php_ini_error_reporting`                | `'E_ALL & ~E_DEPRECATED & ~E_STRICT'`  | Error reporting level       |
| `php_ini_display_errors`                 | `'Off'`                                | Display errors in output    |
| `php_ini_display_startup_errors`         | `'Off'`                                | Display startup errors      |
| `php_ini_log_errors`                     | `'On'`                                 | Log errors to file          |
| `php_ini_date_timezone`                  | `'UTC'`                                | Default timezone            |
| `php_ini_session_gc_maxlifetime`         | `1440`                                 | Session GC lifetime (sec)   |
| `php_ini_opcache_enable`                 | `1`                                    | Enable OPcache              |
| `php_ini_opcache_memory_consumption`     | `128`                                  | OPcache memory (MB)         |
| `php_ini_opcache_max_accelerated_files`  | `10000`                                | OPcache max cached files    |
| `php_ini_opcache_validate_timestamps`    | `1`                                    | Check file timestamps       |
| `php_ini_opcache_revalidate_freq`        | `2`                                    | Revalidation interval (sec) |

### PHP-FPM Pool Configuration (`www.conf`)

Full template deployed for each version with FPM enabled.

| Variable                                   | Default     | Description                     |
|--------------------------------------------|-------------|---------------------------------|
| `php_fpm_pool_user`                        | OS default  | Pool worker user                |
| `php_fpm_pool_group`                       | OS default  | Pool worker group               |
| `php_fpm_pool_listen`                      | auto        | Listen socket (per version)     |
| `php_fpm_pool_listen_owner`                | OS default  | Socket owner                    |
| `php_fpm_pool_listen_group`                | OS default  | Socket group                    |
| `php_fpm_pool_listen_mode`                 | `'0660'`    | Socket permissions              |
| `php_fpm_pool_pm`                          | `'dynamic'` | Process manager mode            |
| `php_fpm_pool_pm_max_children`             | `5`         | Max child processes             |
| `php_fpm_pool_pm_start_servers`            | `2`         | Initial child processes         |
| `php_fpm_pool_pm_min_spare_servers`        | `1`         | Min idle processes              |
| `php_fpm_pool_pm_max_spare_servers`        | `3`         | Max idle processes              |
| `php_fpm_pool_pm_max_requests`             | `0`         | Requests before respawn (0=off) |
| `php_fpm_pool_pm_process_idle_timeout`     | `'10s'`     | Idle timeout (ondemand)         |
| `php_fpm_pool_request_terminate_timeout`   | `0`         | Request timeout (0=off)         |
| `php_fpm_pool_request_slowlog_timeout`     | `0`         | Slowlog timeout (0=off)         |
| `php_fpm_pool_security_limit_extensions`   | `'.php'`    | Allowed script extensions       |

### Composer

| Variable               | Default | Description                         |
|------------------------|---------|-------------------------------------|
| `php_composer_enabled` | `false` | Install Composer (PHP dep. manager) |

## Tags

| Tag             | Scope                             |
|-----------------|-----------------------------------|
| `php`           | All role tasks                    |
| `php:install`   | Installation and verification     |
| `php:service`   | PHP-FPM service management        |
| `php:configure` | Configuration and default version |

## Example Playbook

```yaml
- name: Include php role
  ansible.builtin.include_role:
    name: marcstraube.common.php
  tags:
    - php
  when: php_enabled | default(true) | bool
```

### Multi-Version Example

```yaml
# group_vars/webservers/vars.yml
php_versions:
  - version: '8.3'
    extensions:
      - curl
      - mbstring
      - xml
    fpm: true
  - version: '8.4'
    extensions:
      - curl
      - mbstring
      - xml
      - zip
    fpm: true
    ini:
      memory_limit: '512M'
    fpm_pool:
      pm_max_children: 50

php_default_version: '8.4'
php_composer_enabled: true
```

## Testing

```bash
cd roles/php
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

### Package Naming Per OS

| OS              | Base            | FPM                | Extension            |
|-----------------|-----------------|--------------------|----------------------|
| Arch (official) | `php`           | `php-fpm`          | `php-<ext>`          |
| Arch (AUR)      | `php84`         | `php84-fpm`        | `php84-<ext>`        |
| Debian (Sury)   | `php8.4-cli`    | `php8.4-fpm`       | `php8.4-<ext>`       |
| RHEL (Remi)     | `php84-php-cli` | `php84-php-fpm`    | `php84-php-<ext>`    |

### Arch Linux Version Detection

The role detects the current PHP version in the Arch repos at runtime. If the
requested version matches, official repo packages are used. Otherwise, AUR
packages with the `php<ver>` prefix are installed (e.g., `php84`, `php84-fpm`).

### Arch Extension Mapping (dual map)

Arch ships PHP via two distinct package layouts that disagree on which
extensions are built into the core package, which are split out, and on the
prefix scheme. The role therefore keeps two separate extension maps in
`vars/Archlinux.yml`:

| Map                            | Active when              | Value semantics            |
|--------------------------------|--------------------------|----------------------------|
| `__php_extension_map_official` | requested == current PHP | Concrete package name      |
| `__php_extension_map_aur`      | requested != current PHP | Suffix (prefix is applied) |

Examples:

- `pgsql` → `php-pgsql` (official) / `php84-pgsql` (AUR)
- `xdebug` → `xdebug` (official) / `php84-xdebug` (AUR)
- `pcov` → AUR only (`php84-pcov`); not in the official repos

In both maps an empty string `''` marks an extension as built into the core
package (no separate sub-package installed). Extensions missing from the map
fall back to `<prefix><name>`. A handful of extensions (`redis`, `mongodb`,
`grpc`) are not shipped by the AUR `phpXX` bundle and will hit the fallback —
override `php_versions[].extensions` or supply your own AUR PKGBUILD.

`pcov` is the inverse case: it exists only in the AUR, so it is available on
the AUR path (`php84-pcov`) but not through the official `php` package, and is
therefore absent from `__php_extension_map_official`.

### RHEL Extension Mapping

RHEL uses two maps for the same reason Arch does — the AppStream and Remi
package layouts differ:

| Map                        | `php_redhat_repo` | Value semantics           |
|----------------------------|-------------------|---------------------------|
| `__php_extension_map`      | `appstream`       | Concrete package name     |
| `__php_extension_map_remi` | `remi`            | Suffix (prefix applied)   |

On AppStream, values are full package names (`php-gd`, `php-pecl-xdebug3`).
On Remi (SCL parallel install), values are suffixes applied after the
versioned prefix `php<XX>-php-`, so `xdebug` resolves to
`php84-php-pecl-xdebug3` and `gd` to `php84-php-gd`. Empty string `''` marks
an extension as built into the base `php<XX>-php` package. Remi ships no
unversioned ImageMagick binding — `imagick` maps to `pecl-imagick-im7`
(ImageMagick 7). Version digits in PECL suffixes (`redis6`, `xdebug3`) track
Remi's current packaging and are verified against `rpms.remirepo.net`.

`pcov` is packaged only for Remi (`php<XX>-php-pecl-pcov`), not for AppStream
or EPEL. On the default AppStream path it maps to `''` (a no-op), so requesting
it installs nothing rather than failing; use `php_redhat_repo: 'remi'` to
install it.

### Default Version Handling

| OS     | Mechanism                                       |
|--------|-------------------------------------------------|
| Debian | `update-alternatives`                           |
| RHEL   | Symlink `/usr/local/bin/php` to Remi SCL binary |
| Arch   | Official `php` is default; AUR via symlink      |

### Configuration Approach

| File       | Approach                           | Reason                              |
|------------|------------------------------------|-------------------------------------|
| php.ini    | Drop-in `99-ansible.ini` in conf.d | Upstream untouched, update-safe     |
| `www.conf` | Full template                      | No override mechanism, short enough |

## References

- [Sury PHP repository](https://deb.sury.org)
- [Remi PHP repository](https://rpms.remirepo.net)
- [Remi parallel install guide](https://blog.remirepo.net/post/2024/12/18/Install-PHP-8.4-on-Fedora-RHEL-CentOS-Alma-Rocky-or-other-clone)
- [PHP-FPM configuration](https://www.php.net/manual/en/install.fpm.configuration.php)

## License

MIT

## Author

Marc Straube
