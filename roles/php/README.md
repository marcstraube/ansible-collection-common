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

- ansible-core >= 2.19
- Collections: `community.general`, `kewlfft.aur` (Arch only)
- **Debian/Ubuntu**: Sury PHP repository is set up automatically by this role
- **RHEL/Rocky**: AppStream needs no extra repository; EPEL (PECL extensions
  of the default version) and Remi (`php_redhat_repo: 'remi'`) via
  `package_management` (`dnf_epel_enabled: true`, `dnf_remi_enabled: true`)

## Supported Platforms

| Platform                   | Notes                                                          |
|----------------------------|----------------------------------------------------------------|
| Arch Linux                 | Official repo for current PHP, AUR for legacy versions         |
| Debian Trixie              | Sury repository (multi-version side-by-side)                   |
| EL 9 (Rocky, Alma, RHEL)   | AppStream (default, one version) or Remi (`php_redhat_repo`)   |
| EL 10 (Rocky, Alma, RHEL)  | AppStream (default, one version) or Remi (`php_redhat_repo`)   |

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

#### Available versions

| Platform      | Source              | Versions                                                      |
|---------------|---------------------|---------------------------------------------------------------|
| Arch Linux    | official repository | current release (`auto`)                                      |
| Arch Linux    | AUR                 | older branches as `php<XX>` bundles (e.g. `8.4`)              |
| Debian Trixie | Sury                | every branch Sury builds (`auto` = Debian's own version)      |
| EL 9          | AppStream           | `8.0` (default, `auto`), `8.1`, `8.2`, `8.3` (module streams) |
| EL 10         | AppStream           | `8.3` (default, `auto`, `php-*`), `8.4` (`php8.4-*`)          |
| EL 9 / EL 10  | Remi                | 5.6 through the current stable release, side by side          |

A version the selected source does not carry fails the run with the
available versions and the other source.

#### `php_redhat_repo` — AppStream vs. Remi

| Aspect                | `'appstream'` (default)                          | `'remi'`                                 |
|-----------------------|--------------------------------------------------|------------------------------------------|
| Source                | Distro AppStream (module streams, package sets)  | Remi Collet's third-party repo           |
| Package names         | `php-cli`, `php-fpm`, `php-<ext>` (`php8.4-*`)   | `php<XX>-php-cli`, `php<XX>-php-<ext>`   |
| Paths                 | `/etc/php.d`, `/etc/php-fpm.d`                   | `/etc/opt/remi/php<XX>/...`              |
| Available versions    | EL 9: 8.0–8.3, EL 10: 8.3, 8.4 (one per host)    | 5.6 through current stable, side-by-side |
| Update cadence        | RHEL lifecycle (conservative)                    | Upstream release day                     |
| Lifecycle             | Application-stream support window                | Upstream PHP support per minor           |
| `php_versions` length | Must be 1 (one version per host)                 | Multiple versions allowed                |

With `php_redhat_repo: 'appstream'` the version selects the AppStream
packages:

- EL 9 ships PHP 8.0 as plain packages and 8.1, 8.2 and 8.3 as module
  streams. On a host without PHP the role enables the stream of the
  requested version (`dnf module enable php:<X.Y>`); 8.0 needs no stream.
  When another stream is already enabled, the run fails with the commands
  to switch it (`sudo dnf module reset -y php && sudo dnf module enable -y
  php:<X.Y>`).
- EL 10 ships PHP 8.3 as `php-*` and PHP 8.4 as the package set `php8.4-*`.
  The version selects the package names (`php8.4-cli`, `php8.4-fpm`,
  `php8.4-mbstring`, …); paths and the `php-fpm` service are the same for
  both sets. The two sets conflict, so a host runs one of them.
- `auto` installs the AppStream default of the release (EL 9: 8.0, EL 10:
  8.3). On a host that already has PHP, `auto` resolves to the installed
  version, so a host set up with 8.4 stays idempotent.
- The role never switches a host to another version: when the installed
  version differs from the requested one, the run fails with both versions
  and the commands to remove the installed packages (and to reset the
  stream on EL 9) first.

PECL extensions come from AppStream for every version (`apcu`, `redis`,
`xdebug`, `zip`); the ones only EPEL carries (`imagick`, `memcached`,
`mongodb`) are built against the default version and have no `php8.4-*`
package on EL 10.

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
| `fpm_pools`    | `[]`    | FPM pools of this version (see [PHP-FPM Pools](#php-fpm-pools))      |
| `fpm_pool`     | —       | Deprecated: overrides for the `www` pool, use `fpm_pools`            |

#### How extensions are resolved

`extensions` takes generic extension names. Each platform maps them to its own
package layout in `vars/`:

- an entry with a package name installs that package
- an empty entry means the extension is compiled into the core php package, so
  nothing is installed
- a name absent from the map falls through to `<prefix><name>`

The distinction matters because the platforms disagree. `mbstring` and `pdo` are
separate packages on EL but compiled in on Arch; `openssl`, `pcre`, `session`,
`filter` and `iconv` come with the core packages on Debian and have no package
of that name at all; the PDO drivers ship inside their database package on
Debian, and `pdo_pgsql` does so everywhere (`pgsql` on EL, Remi and Arch).

On Arch there is a second step. Arch installs an extension's `.so` file but
leaves it disabled — `php.ini` keeps the directive commented out and nothing is
written to `conf.d`, not even by separate packages such as `php-gd`. The role
therefore writes `10-ansible-extensions.ini` into each config directory,
carrying an `extension=` line for every configured extension that is not
compiled into the binary. The line follows the listed name, not the package:
`pdo_pgsql` and `pgsql` share a package but each loads only when listed. The
file is rewritten on every run, so dropping an extension from the list removes
its directive.

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
| `php_ini_log_errors`                     | `'On'`                                 | Log errors                  |
| `php_ini_error_log`                      | `'syslog'`                             | Error log destination       |
| `php_ini_expose_php`                     | `'Off'`                                | Expose PHP version header   |
| `php_ini_date_timezone`                  | `'UTC'`                                | Default timezone            |
| `php_ini_session_gc_maxlifetime`         | `1440`                                 | Session GC lifetime (sec)   |
| `php_ini_session_use_strict_mode`        | `1`                                    | Reject client-made IDs      |
| `php_ini_session_cookie_httponly`        | `1`                                    | Hide cookie from JavaScript |
| `php_ini_opcache_enable`                 | `1`                                    | Enable OPcache              |
| `php_ini_opcache_memory_consumption`     | `128`                                  | OPcache memory (MB)         |
| `php_ini_opcache_max_accelerated_files`  | `10000`                                | OPcache max cached files    |
| `php_ini_opcache_validate_timestamps`    | `1`                                    | Check file timestamps       |
| `php_ini_opcache_revalidate_freq`        | `2`                                    | Revalidation interval (sec) |
| `php_ini_extra_settings`                 | `{}`                                   | Extra directives (dict)     |

Three of these defaults are stricter than PHP's own: `php_ini_expose_php`
removes the header naming the exact PHP version, `php_ini_session_use_strict_mode`
makes PHP regenerate session IDs a client invented rather than adopting them,
and `php_ini_session_cookie_httponly` keeps the session cookie out of reach of
JavaScript. Set any of them back to the PHP default through its own variable.

`php_ini_extra_settings` covers directives the variables above do not expose,
for example:

```yaml
php_ini_extra_settings:
  session.save_path: '/var/lib/php/session'
  realpath_cache_size: '4096k'
  zlib.output_compression: 'On'
```

Keys are written verbatim into the drop-in, so dotted directive names work
unchanged. The block is rendered last, so an entry here overrides the same
directive set by one of the variables above. Per-version values belong in
`php_versions[].ini.extra_settings` and are merged on top of the global dict.

#### Error log destination

`php_ini_error_log` defaults to `syslog` rather than a file. The drop-in is
deployed to every SAPI directory, CLI included, so a file path would have to be
writable by every user invoking `php` on the command line — otherwise those runs
lose their errors and warn about the unwritable stream. Logging to syslog has no
such requirement and guarantees that `log_errors` has somewhere to write.

An empty value omits the directive entirely and leaves the destination to the
distribution's own `php.ini`.

For a dedicated file, set it per pool, where FPM runs as a known user:

```yaml
php_fpm_pool_admin_values:
  error_log: '/var/log/php-fpm/www-error.log'
```

Pool directives take precedence over the php.ini drop-in and cannot be
overridden by the application at runtime. Anything PHP writes outside this log —
extension startup warnings, engine-level messages — reaches the FPM master log
only with `php_fpm_pool_catch_workers_output` enabled; FPM discards it otherwise.

#### Log locations

With the default `php_ini_error_log: syslog`, PHP errors land in the journal on
every platform. The FPM master log and any file a pool sets as `error_log` stay
where the package puts them:

| Platform                | FPM log directory                           | Link in `/var/log`     |
|-------------------------|---------------------------------------------|------------------------|
| RHEL (AppStream)        | `/var/log/php-fpm/`                         | —                      |
| RHEL (Remi)             | `/var/opt/remi/php<XX>/log/php-fpm/`        | `/var/log/php<XX>-fpm` |
| Debian (Sury)           | `/var/log/php<X.Y>-fpm.log` (file)          | —                      |
| Arch (official and AUR) | `error_log` in the version's `php-fpm.conf` | —                      |

Remi keeps each version in its own software collection tree, so the role links
the log directory of every version with `fpm: true` to `/var/log/php<XX>-fpm`
(e.g. `/var/log/php84-fpm`). A version without FPM has no such directory and
gets no link. The link is for reading: a pool `error_log` names the real path
below `/var/opt/remi`.

The link of a version dropped from `php_versions` stays: the role does not
uninstall that version, so the link still points to its logs. Remove it by hand
together with the packages (`rm /var/log/php<XX>-fpm`).

### PHP-FPM Pools

Full pool templates deployed for each version with FPM enabled. The
`php_fpm_pool_*` variables are the defaults of every pool.

| Variable                                   | Default     | Description                     |
|--------------------------------------------|-------------|---------------------------------|
| `php_fpm_pool_user`                        | OS default  | Pool worker user                |
| `php_fpm_pool_group`                       | OS default  | Pool worker group               |
| `php_fpm_pool_listen`                      | auto        | Listen address of `www`         |
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
| `php_fpm_pool_catch_workers_output`        | `'no'`      | Capture worker output in log    |
| `php_fpm_pool_admin_values`                | `{}`        | Pool `php_admin_value` mapping  |
| `php_fpm_pool_admin_flags`                 | `{}`        | Pool `php_admin_flag` mapping   |
| `php_fpm_pool_env`                         | `{}`        | Pool `env[...]` mapping         |
| `php_fpm_pools_exclusive`                  | `false`     | Remove unlisted managed pools   |
| `php_fpm_pools_keep`                       | `[]`        | Pool files kept when exclusive  |

#### Named pools

Several applications on one host each get a pool of their own through
`php_versions[].fpm_pools`: their own user, socket permissions for the web
server, `open_basedir` and error log, without a second PHP version. Each
entry is written to `<pool dir>/<name>.conf`; its keys are the
`php_fpm_pool_*` names without the prefix and fall back to the global
defaults. `admin_values`, `admin_flags` and `env` are merged over the global
mappings.

```yaml
php_versions:
  - version: 'auto'
    extensions: ['pdo_pgsql', 'mbstring']
    fpm_pools:
      - name: 'postfixadmin'
        user: 'postfixadmin'
        group: 'postfixadmin'
        listen_owner: 'www-data'
        admin_values:
          open_basedir: '/srv/postfixadmin:/tmp'
      - name: 'roundcube'
        state: 'absent'
```

| Key     | Default     | Description                                                |
|---------|-------------|------------------------------------------------------------|
| `name`  | —           | Pool name and file name, `[a-z0-9_-]+`, unique per version |
| `state` | `'present'` | `absent` removes `<name>.conf` and reloads FPM             |
| `<key>` | global      | Any `php_fpm_pool_*` setting without the prefix            |

- **`www`** is a regular entry that configures `www.conf`. Without one the
  role writes `www.conf` from the global defaults. It cannot be set to
  `absent`: the PHP-FPM package ships the file and may restore it on
  update.
- **Socket:** a named pool listens on a socket of its own next to the `www`
  socket; `php_fpm_pool_listen` addresses the `www` pool only. Set `listen`
  in the entry to choose another address.

  | Platform               | Socket of a named pool                            |
  |------------------------|---------------------------------------------------|
  | Arch Linux (official)  | `/run/php-fpm/php-fpm-<name>.sock`                |
  | Arch Linux (AUR)       | `/run/php<XX>-fpm/php<XX>-fpm-<name>.sock`        |
  | Debian and Ubuntu      | `/run/php/php<X.Y>-fpm-<name>.sock`               |
  | EL (AppStream)         | `/run/php-fpm/<name>.sock`                        |
  | EL (Remi)              | `/var/opt/remi/php<XX>/run/php-fpm/<name>.sock`   |

- **Users:** the pool users and groups must exist before the role runs; the
  role does not create them and fails before it writes a pool file, naming
  the missing accounts. Application users come from the role of the
  application, general groups such as the web server group from
  `marcstraube.common.users`.
- **Configuration test:** after a pool file changed or was removed, the role
  tests the FPM configuration (`php-fpm --test`) before it reloads FPM. When
  the test fails, the role puts every pool file of that version back as it
  was before the run (changed files get their previous content, new files
  are removed, removed files come back) and fails with the test output. The
  running FPM keeps serving the previous configuration and a later restart
  finds it unchanged.
- **AppArmor:** where the packaged `php-fpm` profile is installed, the local
  override grants the sockets of the named pools, including a `listen` path
  outside the default socket directories.
- **SELinux (EL):** sockets in the default socket directory inherit its
  `httpd_var_run_t` context (Remi maps its tree onto the system paths). A
  `listen` path elsewhere needs a matching file context.

#### Pool isolation

All pools of one PHP version run under one FPM master and share its OPcache.
By default OPcache serves a cached script to every pool without checking
file access, so a pool could run a script of another pool that its own user
may not read. For a version with named pools the role therefore sets
`opcache.validate_permission = 1` and `opcache.validate_root = 1` in that
version's `99-ansible.ini`; a version without named pools keeps the drop-in
as before. With these checks a pool cannot include or execute another
pool's files that its user has no permission for.

`open_basedir` is not a security boundary between pools, even with these
checks: a pool can still run a cached script outside its `open_basedir`
once another pool has cached it, as long as its user may read the file.
For real isolation, either disable OPcache in that pool, at the cost of
compiling every script on every request:

```yaml
fpm_pools:
  - name: 'postfixadmin'
    user: 'postfixadmin'
    group: 'postfixadmin'
    admin_flags:
      opcache.enable: 'off'
```

or run the application under an FPM master of its own: another PHP version
or another host.

> **Deprecation:** `php_versions[].fpm_pool` is deprecated and will be removed
> in v4.0.0. While it is set, the role still applies it to the `www` pool and
> prints a deprecation warning. Move its keys into an `fpm_pools` entry named
> `www` instead. A version cannot set both. See #569.

#### Exclusive mode

With `php_fpm_pools_exclusive: true` the role removes pool files from the
pool directory of each version that are neither listed in `fpm_pools` nor in
`php_fpm_pools_keep`, for example after a pool was dropped from the
inventory instead of being set to `absent`. Only files whose content starts
with the `ansible_managed` header are removed; pools shipped by a package or
written by hand never match and are left untouched, and `www.conf` always
counts as listed. If the `ansible_managed` text changes between runs, pools
written with the old text are no longer recognized and survive. List file
names in `php_fpm_pools_keep` to exempt them explicitly.

The mode is opt-in because it deletes files. Review what it would remove
with a check run first:

```bash
ansible-playbook site.yml --tags php:configure --check --diff \
  -e php_fpm_pools_exclusive=true
```

#### Worker environment

PHP-FPM clears the environment of its workers (`clear_env = yes`), so an
application that reads settings through `getenv()` sees none of the host's
variables. `php_fpm_pool_env` writes each entry as `env[NAME] = value` into the
pool; `clear_env` stays on, so only the listed variables reach the workers.
The `env` key of an `fpm_pools` entry is merged over the global mapping.

Example for an outbound proxy:

```yaml
php_fpm_pool_env:
  HTTPS_PROXY: 'http://proxy.example.com:3128'
  NO_PROXY: 'localhost,127.0.0.1,.example.com'
```

Guzzle reads `HTTPS_PROXY` and `NO_PROXY` from the environment. It ignores
`HTTP_PROXY` under a web SAPI, since a request header `Proxy:` would show up
as `HTTP_PROXY` there (httpoxy). libcurl reads the lowercase `http_proxy` and
`https_proxy` itself, which a request header cannot set.

### Composer

| Variable                | Default        | Description                                       |
|-------------------------|----------------|---------------------------------------------------|
| `php_composer_enabled`  | `false`        | Install Composer and its wrappers                 |
| `php_composer_version`  | pinned release | Upstream release where `composer.phar` is used    |
| `php_composer_checksum` | its SHA-256    | Checksum of that `composer.phar` (hash or URL)    |
| `php_composer_proxy`    | `''`           | HTTP(S) proxy for the `composer.phar` download    |

Composer itself does not depend on a PHP version; the interpreter that starts
it decides which version it resolves dependencies for. The role installs one
Composer and puts a wrapper per PHP version into `/usr/local/bin`:

| Wrapper        | Runs on                                                     |
|----------------|-------------------------------------------------------------|
| `composer`     | `php_default_version`                                       |
| `composer<XX>` | each entry in `php_versions` (e.g. `composer84` for `8.4`)  |

Each wrapper starts the CLI binary of its version directly:

| Platform                    | PHP binary                            |
|-----------------------------|---------------------------------------|
| Arch Linux (official `php`) | `/usr/bin/php`                        |
| Arch Linux (AUR `php<XX>`)  | `/usr/bin/php<XX>`                    |
| Debian and Ubuntu (Sury)    | `/usr/bin/php<X.Y>`                   |
| EL (AppStream)              | `/usr/bin/php` (one version per host) |
| EL (Remi)                   | `/usr/bin/php<XX>`                    |

The wrappers are PHP scripts with the binary in the shebang line, not shell
aliases, so scripts, cron jobs and CI pick the version up as well.
`community.general.composer` starts Composer as `php <composer>`; it uses
the default version with the `composer` wrapper, and for another version
takes both paths:

```yaml
- name: Install dependencies on PHP 8.3
  community.general.composer:
    command: install
    working_dir: /srv/app
    executable: /usr/bin/php83
    composer_executable: /usr/local/bin/composer83
```

Composer comes from the source of PHP where that source ships it. The
distribution package depends on the distribution's default PHP, so it is
only installed when that PHP is one of the configured versions; everywhere
else it would pull in a second PHP:

| PHP source                                         | Composer               |
|----------------------------------------------------|------------------------|
| Arch Linux with the official `php` configured      | distribution package   |
| Arch Linux with AUR versions only                  | `composer.phar`        |
| Debian and Ubuntu (Sury, no Composer package)      | `composer.phar`        |
| EL AppStream `php-*` (EL 9, default on EL 10)      | EPEL package           |
| EL 10 AppStream `php8.4-*`                         | `composer.phar`        |
| EL Remi (no Composer package)                      | `composer.phar`        |

The role downloads `composer.phar` from getcomposer.org to
`/usr/local/lib/composer.phar` and verifies it against
`php_composer_checksum`. The download is skipped while the file matches the
checksum; changing `php_composer_version` together with
`php_composer_checksum` replaces it on the next run. A scheduled workflow
proposes both when Composer publishes a new release. To follow upstream
instead, select a channel and its published checksum:

```yaml
php_composer_version: 'latest-stable'
php_composer_checksum: 'sha256:https://getcomposer.org/download/latest-stable/composer.phar.sha256sum'
```

In networks without direct DNS or internet access to getcomposer.org (e.g.
an internal corporate network with an outbound HTTP(S) proxy), set
`php_composer_proxy` to the proxy URL. It is applied as
`HTTPS_PROXY`/`HTTP_PROXY` on the `composer.phar` download only; the
Composer package comes through the package manager's own proxy
configuration. Composer's own requests at runtime (`composer install`,
`composer update`, …) read the proxy from the environment of the user who
runs it.

Composer 2.3 and later needs PHP 7.2.5; for older PHP versions use the LTS
channel `latest-2.2.x`.

Notes:

- A wrapper needs the extensions Composer uses (`phar`, `openssl`, `zip` or
  `unzip`, …) in its version. They are built in everywhere except in the AUR
  `php<XX>` bundles; add them to `php_versions[].extensions` there.
- A Composer package installed earlier on a host that now uses
  `composer.phar`, and the PHP it pulled in, stay installed; the wrappers in
  `/usr/local/bin` take precedence in `PATH`. Remove the package with the
  package manager once nothing else needs it.
- The wrappers replace an existing `/usr/local/bin/composer`. A wrapper of a
  version removed from `php_versions` stays in place.
- `sudo` on EL resets `PATH` to `secure_path`, which leaves out
  `/usr/local/bin`.

### Package Updates

| Variable               | Default     | Description                                                          |
|------------------------|-------------|----------------------------------------------------------------------|
| `php_packages_state`   | `'present'` | `present` installs missing packages only, `latest` updates them      |
| `php_packages_minimum` | `{}`        | Lowest acceptable version per PHP branch, checked after the packages |

A regular run never updates PHP. A targeted update, for example for a
security release, is requested on the command line:

```bash
ansible-playbook site.yml --tags php \
  -e php_packages_state=latest \
  -e '{"php_packages_minimum": {"8.3": "8.3.34"}}'
```

With `latest` the role refreshes the package metadata, updates every
installed package of each configured version in one transaction (including
parts pulled in as dependencies) and restarts the PHP-FPM services whose
packages changed. `php_packages_minimum` then fails the run for every
configured branch whose installed version is still lower, which shows that
the repository does not carry the fix yet; branches that are not configured
on the host are ignored. The check is skipped under `--check`.

On Arch Linux `php_packages_state` is ignored: updating single packages is a
partial upgrade there, PHP updates with the full system upgrade.

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
      extra_settings:
        realpath_cache_size: '8192k'
    fpm_pools:
      - name: 'www'
        pm_max_children: 50

php_default_version: '8.4'
php_composer_enabled: true
```

## Testing

```bash
cd roles/php
molecule test
molecule test -s appstream
molecule test -s remi
molecule test -s pools
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

The `appstream` scenario installs a non-default AppStream version on Rocky
Linux (the `php:8.3` module stream on Rocky 9, the `php8.4-*` package set
on Rocky 10), with Composer from EPEL on Rocky 9 and from `composer.phar`
on Rocky 10; the `remi` scenario installs two Remi versions side by side,
with Composer from `composer.phar`, a named pool under the paths of its
version and the `/var/log/php<XX>-fpm` link of each version. The `pools`
scenario runs named pools on all four platforms: two
pools serve requests under their own users and sockets, a third one is
removed through `state: absent`, the exclusive mode removes only an unlisted
managed pool file, `www.conf` stays the same as without `fpm_pools`, the
deprecated `fpm_pool` prints its warning, a pool change the FPM test rejects
is rolled back while FPM keeps running, and a missing pool user as well as
invalid `fpm_pools` settings are rejected. It also checks that a pool cannot
run a script that only another pool's user may read once that pool has cached
it, and that the OPcache checks are set only while the version has named
pools.

## Notes

### Package Naming Per OS

| OS                          | Base            | FPM                | Extension            |
|-----------------------------|-----------------|--------------------|----------------------|
| Arch (official)             | `php`           | `php-fpm`          | `php-<ext>`          |
| Arch (AUR)                  | `php84`         | `php84-fpm`        | `php84-<ext>`        |
| Debian (Sury)               | `php8.4-cli`    | `php8.4-fpm`       | `php8.4-<ext>`       |
| RHEL (AppStream)            | `php-cli`       | `php-fpm`          | `php-<ext>`          |
| RHEL (AppStream 8.4, EL 10) | `php8.4-cli`    | `php8.4-fpm`       | `php8.4-<ext>`       |
| RHEL (Remi)                 | `php84-php-cli` | `php84-php-fpm`    | `php84-php-<ext>`    |

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

| Map                        | `php_redhat_repo` | Prefix applied                      |
|----------------------------|-------------------|-------------------------------------|
| `__php_extension_map`      | `appstream`       | `php-` (`php8.4-` for 8.4 on EL 10) |
| `__php_extension_map_remi` | `remi`            | `php<XX>-php-`                      |

Both maps hold suffixes that are appended to the prefix of the selected
version. On AppStream `gd` resolves to `php-gd`, or to `php8.4-gd` for
PHP 8.4 on EL 10, and `xdebug` to `php-pecl-xdebug3`. On Remi (SCL
parallel install) the versioned prefix `php<XX>-php-` applies, so `xdebug`
resolves to `php84-php-pecl-xdebug3` and `gd` to `php84-php-gd`. Empty
string `''` marks an extension as built into the base package. Remi ships no
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

| File                       | Approach                           | Reason                              |
|----------------------------|------------------------------------|-------------------------------------|
| php.ini                    | Drop-in `99-ansible.ini` in conf.d | Upstream untouched, update-safe     |
| `www.conf`, `<name>.conf`  | Full template per pool             | No override mechanism, short enough |

## References

- [Sury PHP repository](https://deb.sury.org)
- [Remi PHP repository](https://rpms.remirepo.net)
- [Remi parallel install guide](https://blog.remirepo.net/post/2024/12/18/Install-PHP-8.4-on-Fedora-RHEL-CentOS-Alma-Rocky-or-other-clone)
- [PHP-FPM configuration](https://www.php.net/manual/en/install.fpm.configuration.php)

## License

MIT

## Author

Marc Straube
