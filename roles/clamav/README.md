# marcstraube.common.clamav

Manage ClamAV antivirus (clamd, freshclam, clamav-milter).

## Description

This role installs and configures ClamAV antivirus with clamd on-access
scanning, freshclam signature updates, and optional clamav-milter for mail
server integration. It handles OS-specific differences in package names,
service units, config paths, and user/group settings.

## Requirements

- ansible-core >= 2.19
- EPEL repository on RedHat/Rocky (installed automatically in molecule tests)

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

All variables are defined in `defaults/main.yml` with sensible defaults.

### Role Control

| Variable         | Default | Description            |
|------------------|---------|------------------------|
| `clamav_enabled` | `true`  | Enable the clamav role |

### Package Updates

| Variable                  | Default     | Description                                                      |
|---------------------------|-------------|------------------------------------------------------------------|
| `clamav_packages_state`   | `'present'` | `present` installs missing packages only, `latest` updates them  |
| `clamav_packages_minimum` | `''`        | Lowest acceptable ClamAV version, string or mapping per platform |

A regular run never updates ClamAV. A targeted update, for example for a
security release of the engine, is requested on the command line:

```bash
ansible-playbook site.yml --tags clamav \
  -e clamav_packages_state=latest \
  -e clamav_packages_minimum=1.4.4
```

With `latest` the role refreshes the package metadata and updates every
installed ClamAV package in one transaction, including the parts pulled in
as dependencies (on Debian `clamav*` and `libclamav*`, on EL the EPEL
`clamav*` and `clamd` packages). The role reports the version change.

`clamav_packages_minimum` is checked twice. Before the update the role asks
the package manager which version the update would install and fails
without a change when it is older than the minimum, which shows that the
repository does not carry the fix yet. After the update it checks the
installed version again; this check is skipped under `--check`.

The minimum is an upstream version (`1.4.4`, any distribution release of
it) or the full distribution version (`1.4.3+dfsg-1~deb13u1`,
`1.4.3-1.el9`). Debian and EL ship many fixes as backports that only raise
the release, so only the full version proves such a fix. Versions are
compared with the rules of the distribution (`dpkg --compare-versions`, rpm,
`vercmp`); a minimum without an epoch takes the epoch of the package. For
mixed hosts in one run, pass a mapping per platform; the keys are
`<os_family>-<major>` or `<os_family>`, and a host without a matching key
fails before any change:

```yaml
# clamav-minimum.yml, passed with -e @clamav-minimum.yml
clamav_packages_minimum:
  Debian: '1.4.3+dfsg-1~deb13u1'
  RedHat-9: '1.4.3-1.el9'
  RedHat-10: '1.4.3-1.el10'
  Archlinux: '1.4.4'
```

The packages usually restart clamd, freshclam and the milter themselves.
When they did not, for example because of a `policy-rc.d`, a running
service still executes the replaced binary; the role then restarts each
enabled service that does. A stopped service is not started, and a service
that already runs the new binary is not restarted again.

Signature updates are not part of the package update: freshclam keeps
downloading them on its own schedule.

On Arch Linux `clamav_packages_state` is ignored: updating single packages
is a partial upgrade there, ClamAV updates with the full system upgrade. Run
the role with `latest` together with the system upgrade (`--tags
system-update,clamav`) to restart the services after it.

### Daemon Settings

| Variable                | Default | Description                  |
|-------------------------|---------|------------------------------|
| `clamav_daemon_enabled` | `true`  | Enable clamd scanning daemon |

### Freshclam Settings

| Variable                           | Default               | Description                       |
|------------------------------------|-----------------------|-----------------------------------|
| `clamav_freshclam_enabled`         | `true`                | Enable freshclam database updates |
| `clamav_freshclam_checks_per_day`  | `12`                  | Daily update checks               |
| `clamav_freshclam_database_mirror` | `database.clamav.net` | Update mirror                     |
| `clamav_freshclam_private_mirror`  | `''`                  | Private mirror (overrides public) |
| `clamav_freshclam_http_proxy`      | `''`                  | HTTP proxy server                 |

### Milter

| Variable                | Default | Description                             |
|-------------------------|---------|-----------------------------------------|
| `clamav_milter_enabled` | `false` | Enable clamav-milter (mail integration) |

### Process / Privileges

| Variable       | Default | Description                                     |
|----------------|---------|-------------------------------------------------|
| `clamav_user`  | `''`    | Override OS-specific user (empty = OS default)  |
| `clamav_group` | `''`    | Override OS-specific group (empty = OS default) |

### Socket / Network

| Variable            | Default                  | Description                 |
|---------------------|--------------------------|-----------------------------|
| `clamav_socket`     | `/run/clamav/clamd.sock` | Unix socket path            |
| `clamav_tcp_socket` | `''`                     | TCP port (empty = disabled) |
| `clamav_tcp_addr`   | `127.0.0.1`              | TCP bind address            |

### Scanning Limits

See `defaults/main.yml` for all `clamav_max_*` variables.

### Scan Options

See `defaults/main.yml` for all scan, heuristic, bytecode, and structured data detection options.

### Integration

| Variable                             | Default | Description                      |
|--------------------------------------|---------|----------------------------------|
| `clamav_postfix_integration_enabled` | `false` | Add postfix user to clamav group |
| `clamav_rspamd_integration_enabled`  | `false` | Socket accessible by rspamd      |

### Firewall

| Variable                   | Default    | Description                |
|----------------------------|------------|----------------------------|
| `clamav_firewalld_enabled` | `false`    | Open TCP port in firewalld |
| `clamav_firewalld_zone`    | `'public'` | Firewalld zone             |

### AppArmor

| Variable                  | Default | Description                            |
|---------------------------|---------|----------------------------------------|
| `clamav_apparmor_enabled` | `false` | Enforce AppArmor profile (Arch/Debian) |

### Extra Config

Pass-through for directives not exposed as variables:

```yaml
clamav_extra_config:
  SomeDirective: value
clamav_freshclam_extra_config:
  SomeDirective: value
clamav_milter_extra_config:
  SomeDirective: value
```

### OS-Specific Defaults

| Setting        | Arch Linux             | Debian                 | RedHat/Rocky           |
|----------------|------------------------|------------------------|------------------------|
| User           | clamav                 | clamav                 | clamscan               |
| Daemon service | clamav-daemon.service  | clamav-daemon          | clamd@scan             |
| Config path    | /etc/clamav/clamd.conf | /etc/clamav/clamd.conf | /etc/clamd.d/scan.conf |

## Tags

| Tag                | Scope                               |
|--------------------|-------------------------------------|
| `clamav`           | All role tasks                      |
| `clamav:install`   | Package installation and updates    |
| `clamav:configure` | Configuration files and directories |
| `clamav:service`   | Service management                  |
| `clamav:firewall`  | Firewalld rules                     |
| `clamav:apparmor`  | AppArmor profile enforcement        |

## Example Playbook

```yaml
- name: Include clamav role
  ansible.builtin.include_role:
    name: marcstraube.common.clamav
  tags:
    - security
    - security-extended
    - clamav
  when: clamav_enabled | default(true) | bool
```

### Desktop (on-demand scanning only)

```yaml
clamav_enabled: true
clamav_daemon_enabled: false
clamav_freshclam_enabled: true
```

### Mail Server

```yaml
clamav_enabled: true
clamav_milter_enabled: true
clamav_postfix_integration_enabled: true
clamav_milter_socket_group: 'postfix'
```

## Testing

```bash
cd roles/clamav
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

The role sets NoCOW (`chattr +C`) on the database directory (`/var/lib/clamav`)
when running on BTRFS to reduce copy-on-write overhead from frequent database updates.

## References

- [ClamAV](https://www.clamav.net/) — Open source antivirus engine
- [ClamAV Documentation](https://docs.clamav.net/) — Official documentation and configuration guides

## License

MIT

## Author

Marc Straube
