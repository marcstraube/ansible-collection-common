# marcstraube.common.kubernetes

Install Kubernetes client tooling (kubectl, helm, minikube) with per-user
minikube driver configuration.

## Requirements

- ansible-core >= 2.19
- On Arch Linux: all three tools ship in the `extra` repository (no additional
  setup)
- On Debian: the role adds the Kubernetes (pkgs.k8s.io) and Helm apt
  repositories
- On EL: the role adds the Kubernetes (pkgs.k8s.io) dnf repository; `helm`
  comes from EPEL, which the `package_management` role enables
- minikube needs a container runtime for its driver: the `docker` role with
  the user in `docker_users`, or the `podman` role with the user in
  `podman_rootless_users` (rootless) or `podman_nopasswd_sudo_users`

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

| Variable             | Default | Description                |
|----------------------|---------|----------------------------|
| `kubernetes_enabled` | `false` | Enable the kubernetes role |

### Tools

| Variable                      | Default  | Description                                                   |
|-------------------------------|----------|---------------------------------------------------------------|
| `kubernetes_kubectl_enabled`  | `true`   | Install kubectl                                               |
| `kubernetes_kubectl_version`  | `1.37`   | pkgs.k8s.io release branch on Debian and EL (minor version)   |
| `kubernetes_helm_enabled`     | `true`   | Install helm                                                  |
| `kubernetes_minikube_enabled` | `true`   | Install minikube                                              |
| `kubernetes_minikube_version` | `1.39.0` | Upstream minikube release installed on Debian and EL          |

Arch Linux installs the distribution packages for all three tools and ignores
the version variables. On Debian and EL, kubectl follows the pkgs.k8s.io
repository of the configured minor release, so patch releases arrive through
the package manager; the repository is published per minor version, and
upstream keeps the branches of the supported releases. minikube is installed
from the upstream `.deb`/`.rpm` package pinned to
`kubernetes_minikube_version` and only reinstalled when the installed version
differs. The upstream package carries no signature and no checksum file; the
download relies on HTTPS.

### minikube Driver

| Variable                     | Default  | Description                                                |
|------------------------------|----------|------------------------------------------------------------|
| `kubernetes_minikube_driver` | `docker` | Default driver written to the user config (docker, podman) |

minikube keeps its default driver per user in
`~/.minikube/config/config.json`; there is no system-wide setting. The role
writes that file for every user in `kubernetes_users`. A user may run a second
profile with another driver at any time (`minikube start --driver=podman`).
Both runtimes are provided by other roles:

- `docker`: `marcstraube.common.docker` with the user in `docker_users`
- `podman`: `marcstraube.common.podman` with the user in
  `podman_rootless_users` (rootless mode) or `podman_nopasswd_sudo_users`
  (minikube runs podman through `sudo`)

The kvm2 driver is not offered because no role provides libvirt.

### User Configuration

| Variable                      | Default   | Description                                       |
|-------------------------------|-----------|---------------------------------------------------|
| `kubernetes_users`            | `[]`      | Users receiving a minikube config with the driver |
| `kubernetes_user_config_mode` | `initial` | Default mode when a user entry has no `mode` key  |

Each entry has a `username`, an optional `minikube_driver` overriding
`kubernetes_minikube_driver`, and an optional `mode`:

| Mode       | Behavior                                                         |
|------------|------------------------------------------------------------------|
| `managed`  | Write the config on every run, overwriting later user edits      |
| `initial`  | Write the config only when it is absent, then leave it alone     |
| `disabled` | Skip the user                                                    |

```yaml
kubernetes_users:
  - username: 'johndoe'
    minikube_driver: 'podman'
  - username: 'developer'
    mode: 'managed'
```

## Tags

| Tag                  | Description                          |
|----------------------|--------------------------------------|
| `kubernetes`         | All tasks                            |
| `kubernetes:install` | Repositories and packages            |
| `kubernetes:users`   | Per-user minikube configuration      |

## Example Playbook

```yaml
- hosts: workstations
  roles:
    - role: marcstraube.common.docker
      vars:
        docker_users:
          - 'johndoe'
    - role: marcstraube.common.kubernetes
      vars:
        kubernetes_enabled: true
        kubernetes_users:
          - username: 'johndoe'
```

## Testing

```bash
cd roles/kubernetes
molecule test
```

The scenario installs the tools and the per-user configuration on all four
platforms. Starting a cluster is not part of the scenario; minikube cannot run
inside the test container.

## References

- [kubectl](https://kubernetes.io/docs/reference/kubectl/) and the
  [pkgs.k8s.io repositories](https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/)
- [Helm](https://helm.sh/docs/intro/install/)
- [minikube](https://minikube.sigs.k8s.io/docs/start/) and its
  [drivers](https://minikube.sigs.k8s.io/docs/drivers/)

## License

MIT

## Author

Marc Straube
