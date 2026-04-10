# marcstraube.common.sudo

Configure sudo with security hardening and granular access control.

## Description

Manages the main `/etc/sudoers` file and `/etc/sudoers.d/` drop-in files with
visudo syntax validation. Supports per-user, per-group, and custom sudoers
configurations with optional NOPASSWD and command restrictions.

## Requirements

- ansible-core >= 2.17

## Supported Platforms

| Platform         | Versions |
| ---------------- | -------- |
| Arch Linux       | Rolling  |
| Debian           | Trixie   |
| Rocky Linux / EL | 9, 10    |

## Role Variables

### Role Control

| Variable       | Default | Description             |
| -------------- | ------- | ----------------------- |
| `sudo_enabled` | `true`  | Enable/disable the role |

### Wheel Group

| Variable                       | Default | Description                        |
| ------------------------------ | ------- | ---------------------------------- |
| `sudo_wheel_group`             | `true`  | Allow wheel/sudo group to use sudo |
| `sudo_wheel_requires_password` | `true`  | Require password for wheel group   |

### Users and Groups

| Variable             | Default | Description                                        |
| -------------------- | ------- | -------------------------------------------------- |
| `sudo_users`         | `[]`    | List of users with sudo access (see examples)      |
| `sudo_groups`        | `[]`    | List of groups with sudo access (see examples)     |
| `sudo_sudoers_files` | `[]`    | Custom sudoers.d files (raw content or structured) |

### Security Settings

| Variable                 | Default  | Description                                 |
| ------------------------ | -------- | ------------------------------------------- |
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
| ------------------ | ---------------------------------------------------------- | --------------------------------- |
| `sudo_env_keep`    | `['EDITOR', 'VISUAL', 'LANG', 'LANGUAGE', 'LC_*', 'TERM']` | Environment variables to preserve |
| `sudo_env_reset`   | `true`                                                     | Reset environment variables       |
| `sudo_secure_path` | `'/usr/local/sbin:...'`                                    | Secure PATH for sudo commands     |

### Advanced

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `sudo_mailto` | `''` | Mail recipient for sudo alerts (empty = disabled) |
| `sudo_mail_on` | `[]` | Mail events: badpass, no_host, no_perms, no_user, all_cmnds, always |
| `sudo_editor` | `'/usr/bin/nvim:...'` | Editor for visudo |
| `sudo_always_set_home` | `true` | Always set HOME to target user's home |
| `sudo_extra_defaults` | `[]` | Extra defaults in raw sudoers format |
| `sudo_purge_sudoers_d` | `false` | Remove unmanaged sudoers.d files |
| `sudo_keep_sudoers_files` | `['README']` | Files to keep when purging |

## Examples

### User configuration

```yaml
sudo_users:
  - name: 'admin'
    nopasswd: false
  - name: 'deploy'
    nopasswd: true
    commands:
      - '/usr/bin/systemctl restart myapp'
```

### Group configuration

```yaml
sudo_groups:
  - name: 'developers'
    nopasswd: false
  - name: 'operators'
    nopasswd: true
    commands:
      - '/usr/bin/systemctl *'
```

### Custom sudoers files

```yaml
sudo_sudoers_files:
  # Raw content
  - name: 'backup'
    content: 'backup ALL=(root) NOPASSWD: /usr/bin/rsync'
  # Structured with users and commands
  - name: 'monitoring'
    users:
      - 'nagios'
      - 'zabbix'
    commands:
      - '/usr/lib/nagios/plugins/*'
    nopasswd: true
```

### Example playbook

```yaml
- name: Configure sudo
  hosts: all
  become: true
  roles:
    - role: marcstraube.common.sudo
      tags: [sudo]
```

## Tags

| Tag              | Scope                      |
| ---------------- | -------------------------- |
| `sudo`           | All sudo tasks             |
| `sudo:install`   | Package installation       |
| `sudo:configure` | Sudoers file configuration |

## Testing

```bash
cd roles/sudo
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## License

MIT

## Author

Marc Straube
