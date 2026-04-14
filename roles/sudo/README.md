# marcstraube.common.sudo

Configure sudo with security hardening and granular access control.

## Description

Manages the main `/etc/sudoers` file and `/etc/sudoers.d/` drop-in files with
visudo syntax validation. Supports per-user, per-group, and custom sudoers
configurations with optional NOPASSWD and command restrictions.

## Requirements

- ansible-core >= 2.17
- No additional collections required

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

| Variable       | Default | Description          |
|----------------|---------|----------------------|
| `sudo_enabled` | `true`  | Enable the sudo role |

### Wheel Group

| Variable                       | Default | Description                        |
|--------------------------------|---------|------------------------------------|
| `sudo_wheel_group_enabled`     | `true`  | Allow wheel/sudo group to use sudo |
| `sudo_wheel_requires_password` | `true`  | Require password for wheel group   |

### Users and Groups

| Variable             | Default | Description                                        |
|----------------------|---------|----------------------------------------------------|
| `sudo_users`         | `[]`    | List of users with sudo access (see examples)      |
| `sudo_groups`        | `[]`    | List of groups with sudo access (see examples)     |
| `sudo_sudoers_files` | `[]`    | Custom sudoers.d files (raw content or structured) |

### Security Settings

| Variable                 | Default  | Description                                 |
|--------------------------|----------|---------------------------------------------|
| `sudo_require_tty`       | `false`  | Require TTY for sudo (breaks Ansible/cron)  |
| `sudo_use_pty`           | `true`   | Use PTY for sudo sessions                   |
| `sudo_log_syslog`        | `true`   | Log sudo commands to syslog                 |
| `sudo_log_input`         | `false`  | Log input of sudo sessions                  |
| `sudo_log_output`        | `false`  | Log output of sudo sessions                 |
| `sudo_logfile`           | `''`     | Log to file (empty = syslog only)           |
| `sudo_passwd_timeout`    | `5`      | Password prompt timeout in minutes          |
| `sudo_timestamp_timeout` | `5`      | How long sudo remembers password (minutes)  |
| `sudo_passwd_tries`      | `3`      | Password attempts before lockout            |
| `sudo_pwfeedback`        | `false`  | Show asterisks when typing password         |
| `sudo_lecture`           | `'once'` | Lecture user about sudo (always/never/once) |
| `sudo_insults`           | `false`  | Show insults on wrong password              |

### Environment

| Variable           | Default                                                    | Description                       |
|--------------------|------------------------------------------------------------|-----------------------------------|
| `sudo_env_keep`    | `['EDITOR', 'VISUAL', 'LANG', 'LANGUAGE', 'LC_*', 'TERM']` | Environment variables to preserve |
| `sudo_env_reset`   | `true`                                                     | Reset environment variables       |
| `sudo_secure_path` | `'/usr/local/sbin:...'`                                    | Secure PATH for sudo commands     |

### Advanced

| Variable                       | Default               | Description                      |
|--------------------------------|-----------------------|----------------------------------|
| `sudo_mailto`                  | `''`                  | Mail recipient for alerts        |
| `sudo_mail_on`                 | `[]`                  | Mail events list                 |
| `sudo_editor`                  | `'/usr/bin/nvim:...'` | Editor for visudo                |
| `sudo_always_set_home`         | `true`                | Set HOME to target user's home   |
| `sudo_extra_defaults`          | `[]`                  | Extra raw sudoers defaults       |
| `sudo_purge_sudoers_d_enabled` | `false`               | Remove unmanaged sudoers.d files |
| `sudo_keep_sudoers_files`      | `['README']`          | Files to keep when purging       |

## Tags

| Tag              | Scope                      |
|------------------|----------------------------|
| `sudo`           | All sudo tasks             |
| `sudo:install`   | Package installation       |
| `sudo:configure` | Sudoers file configuration |

## Example Playbook

```yaml
- name: Include sudo role
  ansible.builtin.include_role:
    name: marcstraube.common.sudo
  tags: [sudo]
  when: sudo_enabled | default(true) | bool
```

## Testing

```bash
cd roles/sudo
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

### Deprecated Variables (removed in v2.0.0)

| Old Variable           | New Variable                   |
|------------------------|--------------------------------|
| `sudo_wheel_group`     | `sudo_wheel_group_enabled`     |
| `sudo_purge_sudoers_d` | `sudo_purge_sudoers_d_enabled` |

The old variable names still work as a fallback but will be removed in v2.0.0.
Update your inventory and playbook variables to use the new names.

## License

MIT

## Author

Marc Straube
