# marcstraube.common.python

Install Python infrastructure: Python 3, pip, and pipx.

## Description

Centralize Python/pipx management so downstream roles (ansible, nodejs, etc.)
can rely on a modern pipx being available without each duplicating the
bootstrap logic. On Rocky 9, the EPEL pipx is too old for
`community.general.pipx`, so this role creates a bootstrap venv with a
modern version at `/opt/pipx-bootstrap/bin/pipx`.

## Requirements

- ansible-core >= 2.17
- Collections: `kewlfft.aur` (Arch Linux AUR support)
- EPEL must be enabled on RedHat-family systems before this role runs.
  The `package_management` role handles EPEL installation.

## Supported Platforms

| Platform                   | Notes                                            |
|----------------------------|--------------------------------------------------|
| Arch Linux                 |                                                  |
| Debian Trixie              |                                                  |
| EL 9 (Rocky, Alma, RHEL)  | pipx via bootstrap venv (`/opt/pipx-bootstrap`)  |
| EL 10 (Rocky, Alma, RHEL) |                                                  |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable            | Default | Description              |
|---------------------|---------|--------------------------|
| `python_enabled`    | `true`  | Enable the python role   |

### Python Settings

| Variable                | Default | Description                                 |
|-------------------------|---------|---------------------------------------------|
| `python_pipx_enabled`   | `true`  | Enable pipx for user-level Python tool management |
| `python_extra_packages` | `[]`    | Additional OS packages to install           |

## Tags

| Tag      | Scope          |
|----------|----------------|
| `python` | All role tasks |

## Example Playbook

```yaml
- name: Include python role
  ansible.builtin.include_role:
    name: marcstraube.common.python
  tags:
    - python
  when: python_enabled | default(true) | bool
```

## Testing

```bash
cd roles/python
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

On Rocky 9, the system pipx from EPEL (1.2.1) is too old for
`community.general.pipx`. This role installs pipx via pip into a dedicated
venv at `/opt/pipx-bootstrap/`. Downstream roles should use the bootstrap
path on EL 9.

## License

MIT

## Author

Marc Straube
