# marcstraube.common.sysctl

Configure kernel parameters via sysctl with preset profiles for common use cases.

## Description

Deploys a managed sysctl configuration file to `/etc/sysctl.d/` with support for:

- **Security profile** — CIS benchmark network hardening (IP forwarding, source routing,
  ICMP redirects, SYN cookies, reverse path filtering)
- **Network performance profile** — socket buffers, TCP tuning, connection backlog, TCP Fast Open
- **Database profile** — shared memory, swappiness, dirty page settings, semaphores
- **Web server profile** — connection handling, port range, file descriptors
- **Docker/container profile** — IP forwarding, bridge filtering, inotify limits
- **Custom parameters** — arbitrary key-value pairs

Profiles are composable — enable multiple profiles and they merge together. Custom parameters override profile values.

### Relationship to hardening role

The `hardening` role manages **kernel-level** sysctl (kptr_restrict, ASLR, ptrace, SysRq,
BPF, core dumps, fs.protected_*) in `/etc/sysctl.d/99-hardening.conf`.

This role manages **network-level** sysctl and performance tuning in
`/etc/sysctl.d/90-ansible.conf`. The hardening role's file takes priority (99 > 90).

## Requirements

- ansible-core >= 2.17
- `community.general` collection (for `modprobe` module, Docker profile only)

## Supported Platforms

- Arch Linux
- Debian Trixie
- Rocky Linux 9 / 10

## Role Variables

### Service Control

| Variable         | Default | Description             |
| ---------------- | ------- | ----------------------- |
| `sysctl_enabled` | `true`  | Enable/disable the role |

### Profile Toggles

| Variable                             | Default | Description                                  |
| ------------------------------------ | ------- | -------------------------------------------- |
| `sysctl_profile_security`            | `true`  | CIS network security hardening               |
| `sysctl_profile_network_performance` | `false` | High-traffic network tuning                  |
| `sysctl_profile_database`            | `false` | Database server tuning (PostgreSQL, MariaDB) |
| `sysctl_profile_webserver`           | `false` | Web server tuning (nginx, Apache)            |
| `sysctl_profile_docker`              | `false` | Docker/container host settings               |

### Custom Parameters

| Variable            | Default | Description                                |
| ------------------- | ------- | ------------------------------------------ |
| `sysctl_parameters` | `{}`    | Custom key-value pairs (override profiles) |

### Configuration Options

| Variable                   | Default                         | Description                     |
| -------------------------- | ------------------------------- | ------------------------------- |
| `sysctl_config_file`       | `/etc/sysctl.d/90-ansible.conf` | Destination config file         |
| `sysctl_apply_immediately` | `true`                          | Apply parameters without reboot |

### Profile Parameter Variables

Each profile has a corresponding variable with its parameter set. These can be overridden in inventory:

- `sysctl_security_parameters` — security profile defaults
- `sysctl_network_performance_parameters` — network performance defaults
- `sysctl_database_parameters` — database profile defaults
- `sysctl_webserver_parameters` — web server profile defaults
- `sysctl_docker_parameters` — Docker profile defaults

See `defaults/main.yml` for the full parameter sets.

## Tags

| Tag                | Scope                    |
| ------------------ | ------------------------ |
| `sysctl`           | All role tasks           |
| `sysctl:install`   | Package installation     |
| `sysctl:configure` | Configuration deployment |

## Example Playbook

```yaml
- name: Configure sysctl
  hosts: all
  become: true
  roles:
    - role: marcstraube.common.sysctl
      vars:
        sysctl_profile_security: true
        sysctl_profile_network_performance: true
        sysctl_parameters:
          vm.swappiness: 10
```

## Testing

```bash
cd roles/sysctl
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## License

MIT

## Author

Marc Straube
