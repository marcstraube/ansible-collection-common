# marcstraube.common.ansible

Install Ansible, Molecule testing framework, container drivers and linting
tools.

## Description

This role installs Ansible, Molecule testing framework, container drivers
(Podman/Docker), and linting tools (ansible-lint, yamllint). Installation
methods vary by OS: native packages on Arch Linux, pipx on Debian and
RedHat-family systems.

## Requirements

- ansible-core >= 2.17
- Collections: `community.general`, `kewlfft.aur`
- The `python` role must run before this role to provide pipx.
- On Arch Linux, the `package_management` role must run first to set up the
  AUR helper and `aur_builder` user (required for `molecule-plugins`).
- On RedHat-family systems, EPEL must be enabled (handled by
  `package_management` or this role's fallback task).

## Supported Platforms

| Platform                  | Notes                          |
|---------------------------|--------------------------------|
| Arch Linux                | Native packages via pacman/AUR |
| Debian Trixie             | pipx-based installation        |
| EL 9 (Rocky, Alma, RHEL)  | pipx with bootstrap hack       |
| EL 10 (Rocky, Alma, RHEL) | pipx (system)                  |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable                | Default | Description             |
|-------------------------|---------|-------------------------|
| `ansible_tools_enabled` | `true`  | Enable the ansible role |

### Components

| Variable                         | Default    | Description                                        |
|----------------------------------|------------|----------------------------------------------------|
| `ansible_tools_ansible_enabled`  | `true`     | Enable Ansible install                             |
| `ansible_tools_molecule_enabled` | `true`     | Enable Molecule testing framework install          |
| `ansible_tools_container_driver` | `'podman'` | Container driver (`'podman'`, `'docker'`, or `''`) |
| `ansible_tools_lint_enabled`     | `true`     | Enable linting tools install                       |
| `ansible_tools_extra_plugins`    | `[]`       | Extra pipx-injected plugins (Debian/RedHat)        |

## Tags

| Tag             | Scope          |
|-----------------|----------------|
| `ansible-tools` | All role tasks |

## Example Playbook

```yaml
- name: Include ansible role
  ansible.builtin.include_role:
    name: marcstraube.common.ansible
  tags:
    - ansible-tools
  when: ansible_tools_enabled | default(true) | bool
```

## Testing

```bash
cd roles/ansible
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## References

- [Ansible Documentation](https://docs.ansible.com/) — Official Ansible documentation
- [Molecule](https://ansible.readthedocs.io/projects/molecule/) — Ansible testing framework documentation
- [ansible-lint](https://ansible.readthedocs.io/projects/lint/) — Ansible linting tool documentation

## License

MIT

## Author

Marc Straube
