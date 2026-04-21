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

| Platform                  | Notes                                                  |
|---------------------------|--------------------------------------------------------|
| Arch Linux                | Full support, includes pacman hook                     |
| Debian Trixie             | Full support                                           |
| EL 9 (Rocky, Alma, RHEL)  | Supported (typically disabled on servers with SELinux) |
| EL 10 (Rocky, Alma, RHEL) | Supported (typically disabled on servers with SELinux) |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable           | Default | Description              |
|--------------------|---------|--------------------------|
| `firejail_enabled` | `true`  | Enable the firejail role |

### Application Exclusions

| Variable           | Default                           | Description                           |
|--------------------|-----------------------------------|---------------------------------------|
| `firejail_exclude` | `['keepassxc', 'patch', 'steam']` | Applications excluded from sandboxing |

### Global Settings

| Variable                    | Default          | Description                                                  |
|-----------------------------|------------------|--------------------------------------------------------------|
| `firejail_config`           | *(see defaults)* | Dict of `/etc/firejail/firejail.config` settings             |
| `firejail_apparmor_enabled` | `false`          | Enable AppArmor confinement for all sandboxed apps           |
| `firejail_globals`          | `[]`             | Directives applied to all sandboxed apps via `globals.local` |

### Custom Profiles

| Variable            | Default | Description                                             |
|---------------------|---------|---------------------------------------------------------|
| `firejail_profiles` | `{}`    | Per-app profile overrides as `{app: [directives]}` dict |

### Pacman Hook (Arch Linux Only)

| Variable                       | Default | Description                                         |
|--------------------------------|---------|-----------------------------------------------------|
| `firejail_pacman_hook_enabled` | `true`  | Enable pacman hook to re-run firecfg after upgrades |

## Tags

| Tag                  | Scope                                      |
|----------------------|--------------------------------------------|
| `firejail`           | All role tasks                             |
| `firejail:install`   | Package installation                       |
| `firejail:configure` | Configuration (firecfg, profiles, globals) |

## Example Playbook

```yaml
- name: Include firejail role
  ansible.builtin.include_role:
    name: marcstraube.common.firejail
  tags: [firejail]
  when: firejail_enabled | default(true) | bool
```

## Testing

```bash
cd roles/firejail
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## References

- [Firejail](https://firejail.wordpress.com/) — SUID sandbox program
- [Firejail on GitHub](https://github.com/netblue30/firejail) — Source code and issue tracker

## License

MIT

## Author

Marc Straube
