# Ansible Role: marcstraube.common.apparmor

Configure AppArmor mandatory access control for enhanced security.

## Description

This role installs and configures AppArmor on Arch Linux and Debian systems.
It manages kernel boot parameters (GRUB), profile enforcement, custom profiles,
tunables, and the AppArmor service. RHEL/Rocky systems are not supported as
they use SELinux instead.

AppArmor requires kernel support. On first run (before reboot), profile and
service tasks are skipped gracefully. After reboot with the configured kernel
parameters, all features become active.

**Application-specific profiles** (nginx, postfix, etc.) are not managed by this
role. Each service role manages its own AppArmor profile via a
`<role>_apparmor_enabled` boolean — the service role knows its own binary path
and handles OS differences internally.

## Requirements

- **Ansible**: >= 2.15
- **GRUB bootloader**: Required for kernel parameter configuration
- **Kernel with AppArmor support**: All modern kernels include it

## Supported Platforms

| Platform      | AppArmor Version | Package Source          | Notes                          |
| ------------- | ---------------- | ----------------------- | ------------------------------ |
| Arch Linux    | 4.1.x            | pacman (single package) | Needs GRUB LSM config + reboot |
| Debian Trixie | 4.1.x            | apt (separate packages) | Active by default              |

Both platforms use AppArmor 4.x with ABI 4.0 profile syntax (`include` instead
of `#include`).

### Platform Differences

| Feature | Arch Linux | Debian Trixie |
| ------- | ---------- | ------------- |
| Packaging | Single `apparmor` package | Separate `apparmor`, `-utils`, `-notify`, `-profiles-extra` |
| GRUB parameter | `lsm=landlock,lockdown,yama,integrity,apparmor,bpf` | `apparmor=1 security=apparmor` |
| Active after install | No (reboot required) | Yes |
| CLI tool paths | `/usr/bin/aa-*` | `/usr/sbin/aa-*` |

## Role Variables

### Role Control

| Variable           | Default | Description                    |
| ------------------ | ------- | ------------------------------ |
| `apparmor_enabled` | `true`  | Enable/disable the entire role |

### Boot Configuration

| Variable                | Default                | Description                                     |
| ----------------------- | ---------------------- | ----------------------------------------------- |
| `apparmor_boot_enabled` | `true`                 | Enable AppArmor via GRUB kernel parameters      |
| `apparmor_securityfs`   | `/sys/kernel/security` | Security filesystem path (for kernel detection) |

### Profile Management

| Variable                     | Default                                 | Description                      |
| ---------------------------- | --------------------------------------- | -------------------------------- |
| `apparmor_enforce_profiles`  | `['/usr/sbin/tcpdump', '/usr/bin/man']` | Profiles to set in enforce mode  |
| `apparmor_complain_profiles` | `[]`                                    | Profiles to set in complain mode |
| `apparmor_disable_profiles`  | `[]`                                    | Profiles to disable              |

### Custom Profiles

| Variable                   | Default | Description                                    |
| -------------------------- | ------- | ---------------------------------------------- |
| `apparmor_custom_profiles` | `[]`    | Custom profile definitions (see example below) |

### Tunables

| Variable            | Default | Description                                                   |
| ------------------- | ------- | ------------------------------------------------------------- |
| `apparmor_tunables` | `{}`    | Custom tunables deployed to `/etc/apparmor.d/tunables/custom` |

### Utilities

| Variable                  | Default | Description                                          |
| ------------------------- | ------- | ---------------------------------------------------- |
| `apparmor_utils_enabled`  | `true`  | Install apparmor-utils (aa-status, aa-enforce, etc.) |
| `apparmor_extra_profiles` | `true`  | Install apparmor-profiles-extra                      |
| `apparmor_notify_package` | `true`  | Install apparmor-notify                              |

## Tags

| Tag        | Scope          |
| ---------- | -------------- |
| `apparmor` | All role tasks |

## Example Playbook

```yaml
- name: Configure AppArmor
  hosts: all
  become: true
  roles:
    - role: marcstraube.common.apparmor
      vars:
        apparmor_enforce_profiles:
          - '/usr/sbin/tcpdump'
          - '/usr/bin/man'
        apparmor_tunables:
          HOME: '/home/*'
          HOMEDIRS: '/home'
        apparmor_custom_profiles:
          - name: 'myapp'
            enforce: true
            content: |
              abi <abi/4.0>,
              include <tunables/global>
              /usr/bin/myapp {
                include <abstractions/base>
                /etc/myapp/** r,
                /var/log/myapp/** w,
              }
```

## Integration with Service Roles

This role provides the AppArmor framework. Individual service roles handle their
own profiles using the `<role>_apparmor_enabled` pattern:

```yaml
# Example: nginx role adds its own AppArmor enforcement
nginx_apparmor_enabled: true
```

The service role knows its own binary path (OS-specific) and runs `aa-enforce`
internally. This avoids centralizing OS-specific binary paths in the AppArmor
role.

## Secure Boot / TPM Integration

On Arch Linux systems with Secure Boot, GRUB changes automatically trigger
boot file re-signing (`sbctl sign-all` or HSM signing). If TPM is bound to
PCR 4, the role warns about required TPM re-seal after reboot.

These features are controlled by variables from the `marcstraube.archlinux`
collection (`archlinux_secureboot`, `archlinux_luks_tpm`) and only activate
when those variables are defined.

## Testing

```bash
cd roles/apparmor
molecule test
```

Driver: `vagrant` | Platforms: Arch Linux, Debian Trixie

## License

MIT

## Author

Marc Straube
