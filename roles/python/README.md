# marcstraube.common.python

Install Python infrastructure: Python 3, pip, and pipx.

## Description

Centralize Python/pipx management so downstream roles (ansible, nodejs, etc.)
can rely on a modern pipx being available without each duplicating the
bootstrap logic.

## Requirements

- EPEL must be enabled on RedHat-family systems before this role runs.
  The `package_management` role handles EPEL installation.

## Supported Platforms

| OS            | Python                 | pipx source | Bootstrap needed? |
| ------------- | ---------------------- | ----------- | ----------------- |
| Arch Linux    | System (3.13+)         | pacman      | No                |
| Debian Trixie | System (3.13+)         | apt         | No                |
| Rocky 10      | System (3.12+)         | EPEL        | No                |
| Rocky 9       | python3.12 (AppStream) | pip in venv | Yes               |

On Rocky 9, the EPEL pipx (1.2.1) is too old for `community.general.pipx`.
This role creates `/opt/pipx-bootstrap/bin/pipx` with a modern version.

## Role Variables

| Variable                | Default | Description                       |
| ----------------------- | ------- | --------------------------------- |
| `python_enabled`        | `true`  | Enable/disable the entire role    |
| `python_pipx_enabled`   | `true`  | Install pipx                      |
| `python_extra_packages` | `[]`    | Additional OS packages to install |

## Tags

| Tag      | Scope          |
| -------- | -------------- |
| `python` | All role tasks |

## Example Playbook

```yaml
- name: Include python role
  ansible.builtin.include_role:
    name: marcstraube.common.python
  tags: [base, python]
  when: python_enabled | default(true) | bool
```

## Testing

```bash
cd roles/python
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## License

MIT

## Author

Marc Straube
