# marcstraube.common.base

Base system configuration for all managed hosts.

## Description

Configures foundational system settings: hostname, locale, timezone, keyboard,
kernel with a targeted kernel update and reboot, microcode (Arch), bootloader
(GRUB/systemd-boot), sysctl, and systemd (journald, coredump).

## Requirements

- ansible-core >= 2.19
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

### Login Message

| Variable            | Default | Description                                     |
|---------------------|---------|-------------------------------------------------|
| `base_motd_content` | `''`    | Text of `/etc/motd.d/ansible`; empty removes it |

The message is shown by `pam_motd` on interactive SSH logins. `/etc/motd` and
other files in `/etc/motd.d/` stay untouched.

### Environment

| Variable           | Default | Description                                                         |
|--------------------|---------|---------------------------------------------------------------------|
| `base_environment` | `{}`    | Variables in `/etc/profile.d/base-environment.sh`; empty removes it |

Every login shell that reads `/etc/profile.d` (bash, sh, zsh through its system
profile) exports these variables for all users. fish and systemd services do
not read it; services keep their own settings (e.g. `dnf_proxy`, unit
`Environment=`). Names are used as given and must be valid shell variable
names; values are shell-quoted. Per-user values from the `users` role
(`users-environment.sh`) load later and override these.

For an HTTP proxy, set both spellings, since tools differ in which one they
read:

```yaml
base_environment:
  http_proxy: 'http://proxy.example.com:3128'
  https_proxy: 'http://proxy.example.com:3128'
  no_proxy: 'localhost,127.0.0.1'
  HTTP_PROXY: 'http://proxy.example.com:3128'
  HTTPS_PROXY: 'http://proxy.example.com:3128'
  NO_PROXY: 'localhost,127.0.0.1'
```

### Keyboard

| Variable            | Default        | Description          |
|---------------------|----------------|----------------------|
| `base_keymap`       | `'de-latin1'`  | Console keymap       |
| `base_console_font` | `''`           | Console font         |
| `base_xkb_layout`   | `'de'`         | X11 keyboard layout  |
| `base_xkb_model`    | `''`           | X11 keyboard model   |
| `base_xkb_variant`  | `'nodeadkeys'` | X11 keyboard variant |
| `base_xkb_options`  | `''`           | X11 keyboard options |

### Kernel

| Variable                           | Default       | Description                                             |
|------------------------------------|---------------|---------------------------------------------------------|
| `base_kernel`                      | OS-aware (\*) | Kernel package                                          |
| `base_kernel_extra_packages`       | OS-aware (\*) | Additional kernel packages                              |
| `base_kernel_packages_state`       | `'present'`   | `latest` updates the kernel packages (EL, Debian)       |
| `base_kernel_packages_minimum`     | `''`          | Lowest acceptable kernel version (string or mapping)    |
| `base_kernel_reboot`               | `false`       | Reboot when a newer kernel is installed than running    |
| `base_kernel_reboot_timeout`       | `600`         | Seconds to wait for the host after the reboot           |
| `base_microcode`                   | `'auto'`      | Arch only: auto, intel-ucode, amd-ucode, or ''          |
| `base_kernel_keep_modules_enabled` | `true`        | Arch only: keep running kernel modules after an upgrade |
| `base_mkinitcpio_hooks`            | `[]`          | Arch only: override mkinitcpio hooks                    |
| `base_mkinitcpio_modules`          | `[]`          | Arch only: additional mkinitcpio modules                |
| `base_mkinitcpio_compression`      | `'zstd'`      | Arch only: initramfs compression                        |

(\*) Defaults per platform:

| Platform   | `base_kernel`                                    | `base_kernel_extra_packages` |
|------------|--------------------------------------------------|------------------------------|
| Arch Linux | `linux`                                          | `['linux-headers']`          |
| Debian     | `linux-image-amd64` (`linux-image-arm64` on ARM) | `[]`                         |
| EL         | `kernel`                                         | `[]`                         |

Microcode is managed on Arch Linux only (`auto` selects the package by CPU
vendor).

#### Kernel Updates

A regular run makes sure the kernel package is installed and never updates
it. A kernel security fix is installed with a targeted update:

```bash
ansible-playbook site.yml --tags base:kernel \
  -e base_kernel_packages_state=latest \
  -e base_kernel_packages_minimum=5.14.0-687.65.1.el9_8
```

- `latest` updates every installed kernel package (on EL `kernel`,
  `kernel-core`, `kernel-modules*`, `kernel-devel`, `kernel-tools*`; on Debian
  the image and header metapackages) in one transaction. EL installs the new
  kernel next to the running one and removes the oldest beyond
  `installonly_limit`; Debian moves the metapackage to the new kernel ABI.
  Arch Linux ignores `latest`: the kernel updates with the full system
  upgrade (`--tags system-update,base:kernel`).
- `base_kernel_packages_minimum` is checked before the update (the run fails
  without a change when the repository does not offer it yet) and after it.
  It is either an upstream version (`6.12.115`), which accepts every
  distribution release of it, or the full distribution version
  (`6.12.115-2`, `5.14.0-687.65.1.el9_8`). A mapping per platform covers
  mixed hosts in one run; a host without a matching key fails:

  ```yaml
  # kernel-minimum.yml, passed with -e @kernel-minimum.yml
  base_kernel_packages_minimum:
    Debian: '6.12.115-2'
    RedHat-9: '5.14.0-687.65.1.el9_8'
    RedHat-10: '6.12.0-124.8.1.el10_1'
  ```

- A new kernel takes effect after a reboot. Whenever the role checks the
  kernel (state `latest`, a minimum or `base_kernel_reboot`), it reports a
  pending reboot: the running kernel is not an installed kernel package, or a
  newer kernel is installed. With `-e base_kernel_reboot=true` the role then
  reboots the host and checks that the newest installed kernel runs and meets
  the minimum. The role reboots each host when its turn comes; the order
  across hosts is up to the caller (`--forks 1`, `--limit`), for example so
  that redundant hosts never go down together.

A kernel upgrade on Arch Linux removes the module directory of the running
kernel; until the next reboot nothing that needs a module for the first time
works (for example libvirt cannot create its bridge). With
`base_kernel_keep_modules_enabled` the role installs `kernel-modules-hook`,
whose pacman hooks keep the modules of the running kernel across the upgrade,
and enables `linux-modules-cleanup.service`, which removes the modules of
kernels no longer installed at the next boot. Setting it to `false` disables
the service and leaves the package installed. Debian and EL keep older
kernels installed side by side and are not affected; the setting does nothing
there.

### Bootloader

| Variable                         | Default          | Description                                  |
|----------------------------------|------------------|----------------------------------------------|
| `base_bootloader`                | OS-aware (\*)    | Bootloader: systemd-boot, grub, none         |
| `base_grub_timeout`              | `5`              | GRUB menu timeout (seconds)                  |
| `base_grub_default`              | `'0'`            | Default GRUB menu entry                      |
| `base_kernel_parameters`         | `[]`             | Kernel params for GRUB_CMDLINE_LINUX_DEFAULT |
| `base_systemd_boot_timeout`      | `3`              | systemd-boot loader timeout (seconds)        |
| `base_systemd_boot_console_mode` | `'max'`          | systemd-boot console mode                    |
| `base_systemd_boot_editor`       | `false`          | Allow boot parameter editing (security risk) |

(\*) `base_bootloader` defaults to `grub` on Debian and RHEL-family
distributions, and to `systemd-boot` on Arch Linux (set via
`vars/<OS>.yml`).

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

| Tag                | Scope                                                          |
|--------------------|----------------------------------------------------------------|
| `base`             | All base tasks                                                 |
| `base:hostname`    | Hostname and machine-info                                      |
| `base:locale`      | Locale generation and configuration                            |
| `base:timezone`    | Timezone and NTP                                               |
| `base:motd`        | Login message in /etc/motd.d                                   |
| `base:environment` | Environment variables in /etc/profile.d                        |
| `base:keyboard`    | Console and X11 keyboard                                       |
| `base:packages`    | Package installation/removal                                   |
| `base:sysctl`      | Kernel parameter tuning                                        |
| `base:systemd`     | Journald and coredump configuration                            |
| `base:kernel`      | Kernel, update, reboot; Arch: microcode, modules hook, initrd  |
| `base:bootloader`  | GRUB and systemd-boot configuration                            |

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

## References

- [systemd](https://www.freedesktop.org/wiki/Software/systemd/) — System and service manager
- [locale.conf(5)](https://man7.org/linux/man-pages/man5/locale.conf.5.html) — Locale configuration man page
- [kernel-modules-hook](https://archlinux.org/packages/extra/any/kernel-modules-hook/) — Arch Linux package

## License

MIT

## Author

Marc Straube
