# marcstraube.common.hardware_tokens

Configure hardware security tokens (NitroKey, YubiKey, FIDO2) with udev rules, smart card
daemon, PAM U2F authentication, and screen lock on token removal.

## Description

This role manages OS-level plumbing for hardware tokens:

- **Packages**: CCID driver, libfido2, OpenSC, pcsc-tools, device-specific libraries
- **udev rules**: Device access permissions (NitroKey, YubiKey, generic FIDO2/U2F)
- **pcscd**: PC/SC smart card daemon socket activation
- **PAM U2F**: FIDO2 authentication (2FA or passwordless mode)
- **Lock on removal**: Auto-lock screen when token is unplugged
- **User groups**: plugdev membership on Debian/RHEL (Arch uses uaccess)

GPG smart card configuration (scdaemon.conf) is managed by the
`marcstraube.common.gnupg` role, which deploys per-user `~/.gnupg/scdaemon.conf`
alongside other GnuPG config files.

## Requirements

- ansible-core >= 2.17

## Supported Platforms

| Platform                  | Notes                                      |
|---------------------------|--------------------------------------------|
| Arch Linux                | `uaccess` tag, NitroKey App available      |
| Debian Trixie             | `plugdev` group, NitroKey App not in repos |
| EL 9 (Rocky, Alma, RHEL)  | `plugdev` group, ykman not in repos        |
| EL 10 (Rocky, Alma, RHEL) | `plugdev` group, ykman not in repos        |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable                  | Default | Description                     |
|---------------------------|---------|---------------------------------|
| `hardware_tokens_enabled` | `true`  | Enable the hardware_tokens role |

### Supported Devices

| Variable                                  | Default | Description                                    |
|-------------------------------------------|---------|------------------------------------------------|
| `hardware_tokens_nitrokey_enabled`        | `true`  | Enable NitroKey packages + udev rules          |
| `hardware_tokens_nitrokey_app_enabled`    | `false` | Enable NitroKey App GUI (Arch only)            |
| `hardware_tokens_yubikey_enabled`         | `true`  | Enable YubiKey packages + udev rules           |
| `hardware_tokens_yubikey_manager_enabled` | `false` | Enable ykman CLI (Arch + Debian, not Rocky)    |
| `hardware_tokens_fido2_enabled`           | `true`  | Enable generic FIDO2/U2F packages + udev rules |

### Smartcard / PKCS#11

| Variable                            | Default | Description                   |
|-------------------------------------|---------|-------------------------------|
| `hardware_tokens_smartcard_enabled` | `true`  | Enable pcsc-tools diagnostics |
| `hardware_tokens_opensc_enabled`    | `true`  | Enable OpenSC PKCS#11 tools   |
| `hardware_tokens_pcscd_enabled`     | `true`  | Enable pcscd.socket           |

### PAM U2F

| Variable                              | Default             | Description                                      |
|---------------------------------------|---------------------|--------------------------------------------------|
| `hardware_tokens_pam_u2f_enabled`     | `false`             | Enable PAM U2F module                            |
| `hardware_tokens_pam_u2f_mode`        | `'2fa'`             | `'2fa'` or `'passwordless'`                      |
| `hardware_tokens_pam_u2f_pin_enabled` | `true`              | Enable PIN verification in passwordless mode     |
| `hardware_tokens_pam_u2f_nouserok`    | `true`              | Allow login without registered keys              |
| `hardware_tokens_pam_u2f_system_auth` | `false`             | Configure system-wide PAM instead of per-service |
| `hardware_tokens_pam_u2f_services`    | `['sudo', 'login']` | PAM services (when system_auth is false)         |
| `hardware_tokens_pam_u2f_config`      | see defaults        | authfile, cue, interactive, origin, appid        |

### Udev Rules

| Variable                             | Default | Description                  |
|--------------------------------------|---------|------------------------------|
| `hardware_tokens_udev_rules_enabled` | `true`  | Enable udev rules deployment |

### Lock on Removal

| Variable                                  | Default            | Description                 |
|-------------------------------------------|--------------------|-----------------------------|
| `hardware_tokens_lock_on_removal_enabled` | `false`            | Enable auto-lock on removal |
| `hardware_tokens_lock_on_removal_devices` | Nitrokey 3 + FIDO2 | Vendor/product IDs to watch |

### Users

| Variable                | Default | Description                          |
|-------------------------|---------|--------------------------------------|
| `hardware_tokens_users` | `[]`    | Users to add to device access groups |

## Tags

| Tag                       | Scope                   |
|---------------------------|-------------------------|
| `hardware-tokens`         | All tasks               |
| `hardware-tokens:install` | Package installation    |
| `hardware-tokens:udev`    | udev rules deployment   |
| `hardware-tokens:service` | pcscd socket management |
| `hardware-tokens:users`   | Group membership        |
| `hardware-tokens:pam`     | PAM U2F configuration   |
| `hardware-tokens:lock`    | Lock on removal         |

## Example Playbook

```yaml
- name: Include hardware_tokens role
  ansible.builtin.include_role:
    name: marcstraube.common.hardware_tokens
  tags:
    - hardware-tokens
  when: hardware_tokens_enabled | default(true) | bool
```

## Testing

```bash
cd roles/hardware_tokens
molecule test
```

Driver: `vagrant` | Platforms: Arch Linux, Debian Trixie, Rocky 9

## Notes

- **Arch Linux**: udev rules provided by `nitrokey-udev-rules` package, no manual
  deployment needed. Uses `uaccess` tag instead of `plugdev` group.
- **Debian Trixie**: NitroKey App not available in repos. Uses `plugdev` group.
- **Rocky 9/10**: YubiKey Manager (ykman) not in repos, use pipx if needed. NitroKey
  App not available. Uses `plugdev` group.

## References

- [Yubico Developers](https://developers.yubico.com/) — YubiKey developer documentation and tools
- [Nitrokey Documentation](https://www.nitrokey.com/documentation) — Nitrokey setup and usage guides
- [FIDO2 - FIDO Alliance](https://fidoalliance.org/fido2/) — FIDO2 authentication standard specification

## License

MIT

## Author

Marc Straube
