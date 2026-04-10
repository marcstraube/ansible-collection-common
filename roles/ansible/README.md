# marcstraube.common.ansible

Install Ansible, Molecule testing framework, container drivers and linting
tools.

## Description

This role installs Ansible, Molecule testing framework, container drivers
(Podman/Docker), and linting tools (ansible-lint, yamllint). Installation
methods vary by OS: native packages on Arch Linux, pipx on Debian and
RedHat-family systems.

## Requirements

- **Ansible**: >= 2.17
- The `python` role must run before this role to provide pipx.
- On Arch Linux, the `package_management` role must run first to set up the
  AUR helper and `aur_builder` user (required for `molecule-plugins`).
- On RedHat-family systems, EPEL must be enabled (handled by
  `package_management` or this role's fallback task).

## Supported Platforms

| Platform | Notes |
| -------------- | ----- |
| Arch Linux | Native packages via pacman/AUR |
| Debian Trixie | pipx-based installation |
| Rocky Linux 9 | pipx with bootstrap hack |
| Rocky Linux 10 | pipx (system) |

## Role Variables

| Variable                         | Default    | Description                                        |
| -------------------------------- | ---------- | -------------------------------------------------- |
| `ansible_tools_enabled`          | `true`     | Enable/disable the entire role                     |
| `ansible_tools_install_ansible`  | `true`     | Install Ansible                                    |
| `ansible_tools_install_molecule` | `true`     | Install Molecule                                   |
| `ansible_tools_container_driver` | `'podman'` | Container driver (`'podman'`, `'docker'`, or `''`) |
| `ansible_tools_install_lint`     | `true`     | Install ansible-lint and yamllint                  |
| `ansible_tools_extra_plugins`    | `[]`       | Extra pipx-injected plugins (Debian/RedHat)        |

## Installation Method per OS

| OS                 | Ansible | Molecule + Lint  | Driver                 |
| ------------------ | ------- | ---------------- | ---------------------- |
| Arch Linux         | pacman  | pacman           | AUR (molecule-plugins) |
| Debian Trixie      | apt     | pipx (system)    | pipx inject            |
| Rocky 9            | dnf     | pipx (bootstrap) | pipx inject            |
| Rocky 10+ / Fedora | dnf     | pipx (system)    | pipx inject            |

## Tags

| Tag             | Scope          |
| --------------- | -------------- |
| `ansible-tools` | All role tasks |

## Example Playbook

```yaml
- name: Include ansible tools role
  ansible.builtin.include_role:
    name: marcstraube.common.ansible
  tags: [base, ansible-tools]
  when: ansible_tools_enabled | default(true) | bool
```

## Testing

```bash
cd roles/ansible
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9

## License

MIT

## Author

Marc Straube
