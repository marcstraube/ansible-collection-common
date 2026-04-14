# marcstraube.common.base

Base system configuration for all managed hosts.

## Description

Configures foundational system settings: hostname, locale, timezone, keyboard,
kernel/microcode (Arch), bootloader (GRUB/systemd-boot), sysctl, and systemd
(journald, coredump).

## Requirements

- ansible-core >= 2.17
- Collections: `community.general`, `ansible.posix`

## Supported Platforms

| Platform                  | Notes |
|---------------------------|-------|
| Arch Linux                |       |
| Debian Trixie             |       |
| EL 9 (Rocky, Alma, RHEL)  |       |
| EL 10 (Rocky, Alma, RHEL) |       |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable       | Default | Description          |
|----------------|---------|----------------------|
| `base_enabled` | `true`  | Enable the base role |

### Hostname

| Variable               | Default                    | Description                                          |
|------------------------|----------------------------|------------------------------------------------------|
| `base_hostname`        | `{{ inventory_hostname }}` | System hostname                                      |
| `base_pretty_hostname` | `''`                       | Pretty hostname (shown in UI)                        |
| `base_chassis`         | `'desktop'`                | Chassis type: desktop, laptop, server, vm, container |
| `base_deployment`      | `''`                       | Environment: development, staging, production        |
| `base_icon_name`       | `''`                       | XDG icon name                                        |
| `base_location`        | `''`                       | Physical location description                        |

### Locale

| Variable           | Default                 | Description                   |
|--------------------|-------------------------|-------------------------------|
| `base_locale`      | `'en_US.UTF-8'`         | System locale (LANG)          |
| `base_locales`     | `['en_US.UTF-8 UTF-8']` | Locales to generate           |
| `base_locale_lc_*` | `''`                    | Per-category locale overrides |

### Timezone & NTP

| Variable                    | Default | Description                                   |
|-----------------------------|---------|-----------------------------------------------|
| `base_timezone`             | `'UTC'` | System timezone                               |
| `base_ntp_enabled`          | `false` | Enable systemd-timesyncd (false = use chrony) |
| `base_ntp_servers`          | `[]`    | NTP servers                                   |
| `base_ntp_fallback_servers` | `[]`    | Fallback NTP servers                          |

### Keyboard

| Variable            | Default        | Description          |
|---------------------|----------------|----------------------|
| `base_keymap`       | `'de-latin1'`  | Console keymap       |
| `base_console_font` | `''`           | Console font         |
| `base_xkb_layout`   | `'de'`         | X11 keyboard layout  |
| `base_xkb_model`    | `''`           | X11 keyboard model   |
| `base_xkb_variant`  | `'nodeadkeys'` | X11 keyboard variant |
| `base_xkb_options`  | `''`           | X11 keyboard options |

### Kernel (Arch Linux only)

| Variable                      | Default                         | Description                         |
|-------------------------------|---------------------------------|-------------------------------------|
| `base_kernel`                 | `'linux'`                       | Kernel package                      |
| `base_kernel_extra_packages`  | `['{{ base_kernel }}-headers']` | Additional kernel packages          |
| `base_microcode`              | `'auto'`                        | auto, intel-ucode, amd-ucode, or '' |
| `base_mkinitcpio_hooks`       | `[]`                            | Override mkinitcpio hooks           |
| `base_mkinitcpio_modules`     | `[]`                            | Additional mkinitcpio modules       |
| `base_mkinitcpio_compression` | `'zstd'`                        | Initramfs compression               |

### Bootloader

| Variable                         | Default          | Description                                  |
|----------------------------------|------------------|----------------------------------------------|
| `base_bootloader`                | `'systemd-boot'` | Bootloader: systemd-boot, grub, none         |
| `base_grub_timeout`              | `5`              | GRUB menu timeout (seconds)                  |
| `base_grub_default`              | `'0'`            | Default GRUB menu entry                      |
| `base_kernel_parameters`         | `[]`             | Kernel params for GRUB_CMDLINE_LINUX_DEFAULT |
| `base_systemd_boot_timeout`      | `3`              | systemd-boot loader timeout (seconds)        |
| `base_systemd_boot_console_mode` | `'max'`          | systemd-boot console mode                    |
| `base_systemd_boot_editor`       | `false`          | Allow boot parameter editing (security risk) |

### Packages

| Variable               | Default     | Description                    |
|------------------------|-------------|--------------------------------|
| `base_packages`        | OS-specific | Essential system packages      |
| `base_extra_packages`  | `[]`        | Additional packages to install |
| `base_remove_packages` | `[]`        | Packages to remove             |
| `base_dialog_enabled`  | `true`      | Install dialog package         |

### Sysctl

| Variable      | Default | Description            |
|---------------|---------|------------------------|
| `base_sysctl` | `{}`    | Sysctl key/value pairs |

### Systemd

| Variable                      | Default    | Description                               |
|-------------------------------|------------|-------------------------------------------|
| `base_journald_max_use`       | `'500M'`   | Journald max disk usage                   |
| `base_journald_max_retention` | `'1month'` | Journald max retention                    |
| `base_coredump_storage`       | `'none'`   | Coredump storage: none, external, journal |
| `base_coredump_compress`      | `true`     | Compress coredumps                        |
| `base_coredump_max_size`      | `'2G'`     | Max coredump size                         |
| `base_services_enabled`       | `[]`       | Services to enable and start              |
| `base_services_disabled`      | `[]`       | Services to disable and stop              |

## Tags

| Tag               | Scope                                     |
|-------------------|-------------------------------------------|
| `base`            | All base tasks                            |
| `base:hostname`   | Hostname and machine-info                 |
| `base:locale`     | Locale generation and configuration       |
| `base:timezone`   | Timezone and NTP                          |
| `base:keyboard`   | Console and X11 keyboard                  |
| `base:packages`   | Package installation/removal              |
| `base:sysctl`     | Kernel parameter tuning                   |
| `base:systemd`    | Journald and coredump configuration       |
| `base:kernel`     | Kernel, microcode, mkinitcpio (Arch only) |
| `base:bootloader` | GRUB and systemd-boot configuration       |

## Example Playbook

```yaml
- name: Include base role
  ansible.builtin.include_role:
    name: marcstraube.common.base
  tags:
    - base
  when: base_enabled | default(true) | bool
```

### Server with GRUB and Kernel Parameters

```yaml
- name: Include base role
  ansible.builtin.include_role:
    name: marcstraube.common.base
  vars:
    base_hostname: 'myserver'
    base_timezone: 'Europe/Berlin'
    base_locale: 'en_US.UTF-8'
    base_bootloader: 'grub'
    base_kernel_parameters:
      - 'quiet'
      - 'splash'
  tags:
    - base
  when: base_enabled | default(true) | bool
```

## Testing

```bash
cd roles/base
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

- **Bootloader safety**: GRUB tasks use `lineinfile` to modify `/etc/default/grub`,
  preserving installer-set encryption parameters in `GRUB_CMDLINE_LINUX`.
  `base_kernel_parameters` only affects `GRUB_CMDLINE_LINUX_DEFAULT`.
- **mkinitcpio**: Only managed when explicitly configured. Empty defaults preserve
  the installer-set hooks to avoid breaking encrypted/custom setups.
- **NTP**: Disabled by default -- use the dedicated `chrony` role for NTP.
- Other roles (apparmor, plymouth) append their own kernel parameters using the
  same `lineinfile` pattern on `GRUB_CMDLINE_LINUX_DEFAULT`.
- **Deprecated**: `base_dialog` renamed to `base_dialog_enabled` in v1.x.
  Old variable accepted as fallback, removed in v2.0.0.

## License

MIT

## Author

Marc Straube
