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
- **Debian/Ubuntu**: Sury repository enabled via `package_management`
  (`apt_sury_enabled: true`)
- **RHEL/Rocky**: Remi repository enabled via `package_management`
  (`dnf_remi_enabled: true`)
- **RHEL/Rocky**: EPEL repository enabled for Composer
  (`dnf_epel_enabled: true`)

## Supported Platforms

| Platform                   | Notes |
|----------------------------|-------|
| Arch Linux                 |       |
| Debian Trixie              |       |
| EL 9 (Rocky, Alma, RHEL)   |       |
| EL 10 (Rocky, Alma, RHEL)  |       |

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

| Variable              | Default               | Description                               |
|-----------------------|-----------------------|-------------------------------------------|
| `php_versions`        | `[{version: '8.4'}]`  | PHP versions to install (list of dicts)   |
| `php_default_version` | `'8.4'`               | Default PHP version for CLI               |

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
