# marcstraube.common.docker

Install and configure Docker CE container runtime with daemon management.

## Requirements

- ansible-core >= 2.17
- On Arch Linux: Docker is available in the `extra` repository (no additional setup)
- On Debian/RedHat: The role manages Docker CE repository setup automatically

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

| Variable                 | Default | Description                              |
|--------------------------|---------|------------------------------------------|
| `docker_enabled`         | `true`  | Enable the docker role                   |
| `docker_service_enabled` | `true`  | Enable and start the docker service      |

### Installation

| Variable                 | Default | Description                    |
|--------------------------|---------|--------------------------------|
| `docker_compose_enabled` | `true`  | Install Docker Compose plugin  |
| `docker_buildx_enabled`  | `true`  | Install Docker Buildx plugin   |

### Docker Users

| Variable       | Default | Description                        |
|----------------|---------|------------------------------------|
| `docker_users` | `[]`    | Users to add to the docker group   |

### Daemon Configuration -- Storage

| Variable                | Default           | Description                |
|-------------------------|-------------------|----------------------------|
| `docker_data_root`      | `/var/lib/docker` | Docker data root directory |
| `docker_storage_driver` | `overlay2`        | Storage driver             |
| `docker_storage_opts`   | `[]`              | Storage driver options     |

### Daemon Configuration -- Logging

| Variable            | Default    | Description                               |
|---------------------|------------|-------------------------------------------|
| `docker_log_driver` | `journald` | Default log driver                        |
| `docker_log_opts`   | `{}`       | Log driver options                        |
| `docker_log_level`  | `''`       | Daemon log level (empty = Docker default) |

### Daemon Configuration -- Network

| Variable                       | Default | Description                              |
|--------------------------------|---------|------------------------------------------|
| `docker_bip`                   | `''`    | Bridge IP CIDR                           |
| `docker_default_address_pools` | `[]`    | Custom address pools                     |
| `docker_dns`                   | `[]`    | DNS servers for containers               |
| `docker_dns_search`            | `[]`    | DNS search domains                       |
| `docker_iptables`              | `true`  | Allow Docker to manage iptables rules    |
| `docker_ipv6`                  | `false` | Enable IPv6 support                      |
| `docker_mtu`                   | `0`     | Bridge MTU (0 = auto)                    |
| `docker_userland_proxy`        | `true`  | Use userland proxy for loopback traffic  |

### Daemon Configuration -- Security

| Variable                    | Default | Description                                      |
|-----------------------------|---------|--------------------------------------------------|
| `docker_live_restore`       | `true`  | Keep containers running during daemon restart    |
| `docker_no_new_privileges`  | `false` | Set no-new-privileges default                    |
| `docker_selinux_enabled`    | `false` | Enable SELinux support                           |
| `docker_userns_remap`       | `''`    | User namespace remapping                         |
| `docker_icc`                | `true`  | Allow inter-container communication              |

### Daemon Configuration -- Registry

| Variable                      | Default | Description                |
|-------------------------------|---------|----------------------------|
| `docker_registry_mirrors`     | `[]`    | Registry mirrors           |
| `docker_insecure_registries`  | `[]`    | Allow insecure registries  |

### Daemon Configuration -- Runtime

| Variable                          | Default | Description                              |
|-----------------------------------|---------|------------------------------------------|
| `docker_default_runtime`          | `runc`  | Default OCI runtime                      |
| `docker_init`                     | `false` | Run init process in containers           |
| `docker_default_ulimits`          | `{}`    | Default ulimits                          |
| `docker_shutdown_timeout`         | `0`     | Graceful shutdown timeout (0 = default)  |
| `docker_max_concurrent_downloads` | `0`     | Max concurrent downloads (0 = default)   |
| `docker_max_concurrent_uploads`   | `0`     | Max concurrent uploads (0 = default)     |

### Daemon Configuration -- Extra

| Variable                | Default | Description                              |
|-------------------------|---------|------------------------------------------|
| `docker_daemon_options` | `{}`    | Free-form dict merged into daemon.json   |

### Systemd Integration

| Variable                | Default  | Description                           |
|-------------------------|----------|---------------------------------------|
| `docker_socket_enabled` | `true`   | Enable docker socket                  |
| `docker_prune_enabled`  | `false`  | Enable system prune timer             |
| `docker_prune_schedule` | `weekly` | Prune schedule (OnCalendar format)    |
| `docker_prune_all`      | `true`   | Prune all unused images               |
| `docker_prune_volumes`  | `true`   | Prune volumes during prune            |

## Tags

| Tag                | Description                                                |
|--------------------|------------------------------------------------------------|
| `docker`           | All docker tasks                                           |
| `docker:install`   | Repository setup, package installation, BTRFS optimization |
| `docker:configure` | Daemon configuration and user group management             |
| `docker:service`   | Systemd service, socket, and prune timer management        |

## Example Playbook

```yaml
- name: Docker hosts
  hosts: docker_hosts
  roles:
    - role: marcstraube.common.docker
      vars:
        docker_users:
          - 'johndoe'
        docker_live_restore: true
        docker_log_driver: 'journald'
        docker_registry_mirrors:
          - 'https://mirror.gcr.io'
```

## Testing

```bash
cd collections/ansible_collections/marcstraube/common
molecule test -s docker -p docker-archlinux
```

## References

- [Docker CE Installation](https://docs.docker.com/engine/install/)
- [Docker Daemon Configuration](https://docs.docker.com/reference/cli/dockerd/#daemon-configuration-file)
- [Docker Storage Drivers](https://docs.docker.com/engine/storage/drivers/)
- [Docker Logging Drivers](https://docs.docker.com/engine/logging/configure/)

## License

MIT

## Author

Marc Straube
