# marcstraube.common.podman

Install and configure Podman container runtime with rootless support and Docker compatibility.

## Description

This role installs and configures Podman as a container runtime with:

- Full Docker CLI compatibility (aliases, socket symlink, podman-docker package)
- Rootless container support with subuid/subgid and systemd linger
- Configurable registries (V2 format), storage, and container defaults
- Systemd integration (socket activation, auto-update timer, system prune timer)
- BTRFS optimizations (NoCOW + subvolume for container storage)
- Security defaults (seccomp, capabilities, user namespaces, SELinux/AppArmor)

## Requirements

- ansible-core >= 2.17
- No additional collections required

## Supported Platforms

| Platform      | Podman Version | Notes                            |
| ------------- | -------------- | -------------------------------- |
| Arch Linux    | 5.8+           | Rolling release, latest upstream |
| Debian Trixie | 5.4+           | Stable                           |
| Rocky 9       | 5.0+           | AppStream                        |
| Rocky 10      | 5.6+           | AppStream                        |

## Role Variables

### Service Control

| Variable         | Default | Description             |
| ---------------- | ------- | ----------------------- |
| `podman_enabled` | `true`  | Enable/disable the role |

### Installation

| Variable                 | Default             | Description                                             |
| ------------------------ | ------------------- | ------------------------------------------------------- |
| `podman_compose_enabled` | `true`              | Install podman-compose                                  |
| `podman_docker_enabled`  | `true`              | Install podman-docker package                           |
| `podman_podlet_enabled`  | `false`             | Install podlet (generate Quadlet files from containers) |
| `podman_desktop_enabled` | `false`             | Install Podman Desktop GUI (Arch Linux only, AUR)       |
| `podman_tools`           | `[buildah, skopeo]` | Additional container tools                              |

### Docker Compatibility

| Variable                       | Default | Description                                |
| ------------------------------ | ------- | ------------------------------------------ |
| `podman_docker_alias`          | `true`  | Create docker/docker-compose shell aliases |
| `podman_docker_socket_symlink` | `true`  | Create docker.sock symlink                 |

### Rootless Configuration

| Variable                | Default | Description                            |
| ----------------------- | ------- | -------------------------------------- |
| `podman_rootless_users` | `[]`    | Users to configure for rootless podman |
| `podman_subid_count`    | `65536` | Default subuid/subgid range size       |

### Registries (registries.conf V2)

| Variable                    | Default                         | Description                          |
| --------------------------- | ------------------------------- | ------------------------------------ |
| `podman_registries_search`  | `[docker.io, quay.io, ghcr.io]` | Unqualified search registries        |
| `podman_short_name_mode`    | `permissive`                    | Short-name resolution mode           |
| `podman_registries`         | `[]`                            | Registry configurations with mirrors |
| `podman_registries_blocked` | `[]`                            | Blocked registries                   |
| `podman_registries_auth`    | `[]`                            | Registry authentication credentials  |

### Storage (storage.conf)

| Variable                  | Default                                    | Description                               |
| ------------------------- | ------------------------------------------ | ----------------------------------------- |
| `podman_storage_driver`   | `overlay`                                  | Storage driver (overlay, vfs, btrfs, zfs) |
| `podman_storage_root`     | `/var/lib/containers/storage`              | Root storage location                     |
| `podman_storage_rootless` | `$HOME/.local/share/containers/storage`    | Rootless storage path                     |
| `podman_storage_options`  | `{overlay: {mountopt: nodev,metacopy=on}}` | Storage driver options                    |

### Network

| Variable                      | Default        | Description                                           |
| ----------------------------- | -------------- | ----------------------------------------------------- |
| `podman_network_backend`      | `netavark`     | Network backend (netavark, cni)                       |
| `podman_default_subnet`       | `10.88.0.0/16` | Default subnet                                        |
| `podman_firewall_driver`      | `firewalld`    | Firewall driver (firewalld, nftables, iptables, none) |
| `podman_rootless_network_cmd` | `pasta`        | Rootless network command (pasta, slirp4netns)         |

### Container Defaults

| Variable                      | Default                 | Description                      |
| ----------------------------- | ----------------------- | -------------------------------- |
| `podman_runtime`              | `crun`                  | OCI runtime (crun, runc)         |
| `podman_default_capabilities` | Podman defaults         | Default container capabilities   |
| `podman_default_ulimits`      | `{nofile: 65536:65536}` | Default ulimits                  |
| `podman_default_pids_limit`   | `2048`                  | PID limit per container          |
| `podman_log_driver`           | `journald`              | Container log driver             |
| `podman_userns_mode`          | `auto`                  | User namespace mode              |
| `podman_label`                | `true`                  | SELinux/MAC container separation |
| `podman_seccomp_enabled`      | `true`                  | Enable seccomp profile           |

### Systemd Integration

| Variable                     | Default  | Description                 |
| ---------------------------- | -------- | --------------------------- |
| `podman_socket_enabled`      | `true`   | Enable podman API socket    |
| `podman_auto_update_enabled` | `false`  | Enable auto-update timer    |
| `podman_prune_enabled`       | `false`  | Enable system prune timer   |
| `podman_prune_schedule`      | `weekly` | Prune schedule (OnCalendar) |

### BTRFS

| Variable               | Default | Description                                |
| ---------------------- | ------- | ------------------------------------------ |
| `podman_btrfs_enabled` | `true`  | Enable BTRFS NoCOW + subvolume for storage |

## Tags

| Tag                | Scope                         |
| ------------------ | ----------------------------- |
| `podman`           | All podman tasks              |
| `podman:install`   | Package installation          |
| `podman:configure` | Configuration files and BTRFS |
| `podman:rootless`  | Rootless user setup           |
| `podman:service`   | Systemd services and timers   |

## Example Playbook

```yaml
- name: Configure podman
  hosts: all
  become: true
  tasks:
    - name: Include podman role
      ansible.builtin.include_role:
        name: marcstraube.common.podman
      tags: [podman]
      when: podman_enabled | default(true) | bool
```

## Example Inventory

```yaml
# group_vars/all/vars.yml
podman_enabled: true

# group_vars/workstations/vars.yml
podman_rootless_users:
  - username: 'developer'
    subuid_start: 100000
    subuid_count: 65536
    subgid_start: 100000
    subgid_count: 65536
    linger: true
```

## Testing

```bash
cd roles/podman
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## License

MIT

## Author

Marc Straube
