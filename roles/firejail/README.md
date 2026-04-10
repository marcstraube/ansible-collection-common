# marcstraube.common.firejail

Configure Firejail application sandboxing.

## Description

Installs and configures Firejail for application sandboxing via `firecfg`. Manages
global settings, application exclusions, per-application profile overrides, and
optional AppArmor confinement. On Arch Linux, installs a pacman hook to re-run
`firecfg` after package installations.

## Requirements

- ansible-core >= 2.17
- No additional collections required

## Supported Platforms

| Platform       | Notes                                                  |
| -------------- | ------------------------------------------------------ |
| Arch Linux     | Full support, includes pacman hook                     |
| Debian Trixie  | Full support                                           |
| Rocky Linux 9  | Supported (typically disabled on servers with SELinux) |
| Rocky Linux 10 | Supported (typically disabled on servers with SELinux) |

## Role Variables

### Role Control

| Variable           | Default | Description             |
| ------------------ | ------- | ----------------------- |
| `firejail_enabled` | `true`  | Enable/disable the role |

### Application Exclusions

| Variable           | Default                           | Description                           |
| ------------------ | --------------------------------- | ------------------------------------- |
| `firejail_exclude` | `['keepassxc', 'patch', 'steam']` | Applications excluded from sandboxing |

### Global Settings

| Variable            | Default          | Description                                                  |
| ------------------- | ---------------- | ------------------------------------------------------------ |
| `firejail_config`   | *(see defaults)* | Dict of `/etc/firejail/firejail.config` settings             |
| `firejail_apparmor` | `false`          | Enable AppArmor confinement for all sandboxed apps           |
| `firejail_globals`  | `[]`             | Directives applied to all sandboxed apps via `globals.local` |

### Custom Profiles

| Variable            | Default | Description                                             |
| ------------------- | ------- | ------------------------------------------------------- |
| `firejail_profiles` | `{}`    | Per-app profile overrides as `{app: [directives]}` dict |

### Pacman Hook (Arch Linux Only)

| Variable               | Default | Description                                          |
| ---------------------- | ------- | ---------------------------------------------------- |
| `firejail_pacman_hook` | `true`  | Install pacman hook to re-run firecfg after upgrades |

## Tags

| Tag                  | Scope                                      |
| -------------------- | ------------------------------------------ |
| `firejail`           | All role tasks                             |
| `firejail:install`   | Package installation                       |
| `firejail:configure` | Configuration (firecfg, profiles, globals) |

## Example Playbook

```yaml
- name: Configure firejail
  hosts: workstations
  become: true
  tasks:
    - name: Include firejail role
      ansible.builtin.include_role:
        name: marcstraube.common.firejail
      tags: [firejail]
      when: firejail_enabled | default(true) | bool
```

### Custom Profiles with AppArmor

```yaml
- name: Include firejail role
  ansible.builtin.include_role:
    name: marcstraube.common.firejail
  vars:
    firejail_apparmor: true
    firejail_exclude:
      - 'keepassxc'
      - 'steam'
    firejail_profiles:
      firefox:
        - 'whitelist ${HOME}/.mozilla'
        - 'private-tmp'
      chromium:
        - 'whitelist ${HOME}/.config/chromium'
        - 'private-tmp'
```

## Testing

```bash
cd roles/firejail
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## License

MIT

## Author

Marc Straube
