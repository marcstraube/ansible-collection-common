# marcstraube.common.podman

Install and configure Podman container runtime with rootless support and Docker compatibility.

## Requirements

- ansible-core >= 2.17
- `kewlfft.aur` collection (for AUR packages on Arch Linux)

## Supported Platforms

| Platform                    | Notes |
|-----------------------------|-------|
| Arch Linux                  |       |
| Debian Trixie               |       |
| EL 9 (Rocky, Alma, RHEL)    |       |
| EL 10 (Rocky, Alma, RHEL)   |       |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable         | Default | Description             |
|------------------|---------|-------------------------|
| `podman_enabled` | `true`  | Enable the podman role  |

### Installation

| Variable                 | Default             | Description                                             |
|--------------------------|---------------------|---------------------------------------------------------|
| `podman_compose_enabled` | `true`              | Install podman-compose                                  |
| `podman_docker_enabled`  | `true`              | Install podman-docker package                           |
| `podman_podlet_enabled`  | `false`             | Install podlet (generate Quadlet files from containers) |
| `podman_desktop_enabled` | `false`             | Install Podman Desktop GUI (Arch Linux only, AUR)       |
| `podman_tools`           | `[buildah, skopeo]` | Additional container tools                              |

### Docker Compatibility

| Variable                                 | Default | Description                                |
|------------------------------------------|---------|--------------------------------------------|
| `podman_docker_alias_enabled`            | `true`  | Enable docker/docker-compose shell aliases |
| `podman_docker_socket_symlink_enabled`   | `true`  | Enable docker.sock symlink                 |

### Rootless Configuration

| Variable                | Default | Description                            |
|-------------------------|---------|----------------------------------------|
| `podman_rootless_users` | `[]`    | Users to configure for rootless podman |
| `podman_subid_count`    | `65536` | Default subuid/subgid range size       |

### Registries (registries.conf V2)

| Variable                    | Default                          | Description                          |
|-----------------------------|----------------------------------|--------------------------------------|
| `podman_registries_search`  | `[docker.io, quay.io, ghcr.io]`  | Unqualified search registries        |
| `podman_short_name_mode`    | `permissive`                     | Short-name resolution mode           |
| `podman_registries`         | `[]`                             | Registry configurations with mirrors |
| `podman_registries_blocked` | `[]`                             | Blocked registries                   |
| `podman_registries_auth`    | `[]`                             | Registry authentication credentials  |

### Storage (storage.conf)

| Variable                  | Default                                 | Description                               |
|---------------------------|-----------------------------------------|-------------------------------------------|
| `podman_storage_driver`   | `overlay`                               | Storage driver (overlay, vfs, btrfs, zfs) |
| `podman_storage_root`     | `/var/lib/containers/storage`           | Root storage location                     |
| `podman_storage_rootless` | `$HOME/.local/share/containers/storage` | Rootless storage path                     |
| `podman_storage_options`  | `{overlay: {mountopt: ...}}`            | Storage driver options                    |

### Network

| Variable                      | Default        | Description                                           |
|-------------------------------|----------------|-------------------------------------------------------|
| `podman_network_backend`      | `netavark`     | Network backend (netavark, cni)                       |
| `podman_default_subnet`       | `10.88.0.0/16` | Default subnet                                        |
| `podman_firewall_driver`      | `firewalld`    | Firewall driver (firewalld, nftables, iptables, none) |
| `podman_rootless_network_cmd` | `pasta`        | Rootless network command (pasta, slirp4netns)         |

### Container Defaults

| Variable                      | Default         | Description                      |
|-------------------------------|-----------------|----------------------------------|
| `podman_runtime`              | `crun`          | OCI runtime (crun, runc)         |
| `podman_default_capabilities` | Podman defaults | Default container capabilities   |
| `podman_default_ulimits`      | `{nofile: ...}` | Default ulimits                  |
| `podman_default_pids_limit`   | `2048`          | PID limit per container          |
| `podman_log_driver`           | `journald`      | Container log driver             |
| `podman_userns_mode`          | `auto`          | User namespace mode (see below)  |
| `podman_label`                | `true`          | SELinux/MAC container separation |
| `podman_seccomp_enabled`      | `true`          | Enable seccomp profile           |

### User Namespace Modes (`podman_userns_mode`)

Controls UID/GID mapping between host and containers via `userns` in
`containers.conf`. Available modes:

| Mode      | Description                                                        |
|-----------|--------------------------------------------------------------------|
| `auto`    | Automatic UID remapping per container — best isolation (default)   |
| `host`    | No remapping — container root = host root (or host user for rootless) |
| `keep-id` | Map host UID into container (rootless only)                        |
| `nomap`   | Do not map additional UIDs into the user namespace                 |

**When to override per container:** Some images (e.g., Uptime Kuma) run `chown`
in their entrypoint, which fails under `auto` because container root is mapped
to a high subuid on the host. These containers need `--userns=host` at runtime,
passed via Quadlet or `podman run`. The system-wide default should remain `auto`
for security — override only per container where required.

### Systemd Integration

| Variable                     | Default  | Description                 |
|------------------------------|----------|-----------------------------|
| `podman_socket_enabled`      | `true`   | Enable podman API socket    |
| `podman_auto_update_enabled` | `false`  | Enable auto-update timer    |
| `podman_prune_enabled`       | `false`  | Enable system prune timer   |
| `podman_prune_schedule`      | `weekly` | Prune schedule (OnCalendar) |

## Tags

| Tag                | Scope                          |
|--------------------|--------------------------------|
| `podman`           | All podman tasks               |
| `podman:install`   | Package installation and BTRFS |
| `podman:configure` | Configuration files            |
| `podman:rootless`  | Rootless user setup            |
| `podman:service`   | Systemd services and timers    |

## Example Playbook

```yaml
- name: Configure podman
  hosts: all
  become: true
  tasks:
    - name: Include podman role
      ansible.builtin.include_role:
        name: marcstraube.common.podman
      tags:
        - podman
      when: podman_enabled | default(true) | bool
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
