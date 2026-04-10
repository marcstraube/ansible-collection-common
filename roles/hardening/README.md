# marcstraube.common.hardening

Kernel hardening (sysctl), kernel module blacklisting, and filesystem mount
hardening.

## Description

This role handles three hardening areas:

- **Kernel sysctl** — Security-focused kernel parameters via `/etc/sysctl.d/`
- **Module blacklisting** — Disable unused filesystem and network kernel modules via `/etc/modprobe.d/`
- **Filesystem mounts** — Harden mount options for `/tmp`, `/dev/shm`, `/var/tmp`

**Not in scope** (handled by other roles):

| Area                                         | Role                                         |
| -------------------------------------------- | -------------------------------------------- |
| Network sysctl (ip_forward, rp_filter, etc.) | `marcstraube.common.sysctl`                  |
| Audit daemon and rules                       | `marcstraube.common.auditd`                  |
| AppArmor                                     | `marcstraube.common.apparmor`                |
| Firejail sandboxing                          | `marcstraube.common.firejail`                |
| PAM hardening                                | `marcstraube.common.pam_hardening`           |
| GRUB kernel command line                     | `marcstraube.common.base` (bootloader tasks) |

## Requirements

- `ansible.posix` collection (for `ansible.posix.mount`)

## Supported Platforms

- Arch Linux
- Debian Trixie (13)
- Rocky Linux 9
- Rocky Linux 10

## GRUB Hardening Parameters

These kernel command line parameters complement this role's sysctl settings.
Configure them via the base role's bootloader variables:

| Parameter                  | Effect                                                        |
| -------------------------- | ------------------------------------------------------------- |
| `lockdown=integrity`       | Prevent modification of running kernel (modules, /dev/mem)    |
| `lockdown=confidentiality` | Additionally block reading kernel memory (breaks hibernation) |
| `init_on_alloc=1`          | Zero memory at allocation (~1% perf cost)                     |
| `init_on_free=1`           | Zero memory at free (~5% perf cost, skip on DB servers)       |
| `vsyscall=none`            | Disable legacy vsyscall (safe on modern glibc 2.14+)          |
| `debugfs=off`              | Prevent mounting debugfs                                      |
| `page_alloc.shuffle=1`     | Randomize page allocator free lists                           |
| `iommu=force`              | Force IOMMU isolation (5-15% I/O cost, skip in VMs)           |

## Role Variables

### Role Control

| Variable                    | Default | Description                       |
| --------------------------- | ------- | --------------------------------- |
| `hardening_enabled`         | `true`  | Master toggle                     |
| `hardening_kernel_enabled`  | `true`  | Enable sysctl hardening           |
| `hardening_modules_enabled` | `true`  | Enable module blacklisting        |
| `hardening_fs_enabled`      | `true`  | Enable filesystem mount hardening |

### Kernel Sysctl

| Variable                               | Default  | Description                                              |
| -------------------------------------- | -------- | -------------------------------------------------------- |
| `hardening_kernel_settings`            | *(dict)* | Sysctl key-value pairs (see defaults)                    |
| `hardening_kernel_perf_event_paranoid` | `true`   | Deploy perf_event_paranoid (3 on Arch/Debian, 2 on RHEL) |
| `hardening_kernel_io_uring_disabled`   | `2`      | io_uring restriction (0/1/2). Kernel >= 6.6 only         |

### Module Blacklisting

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `hardening_modules_blacklist_fs` | cramfs, freevxfs, hfs, hfsplus, jffs2, udf | Filesystem modules to blacklist |
| `hardening_modules_blacklist_net` | dccp, sctp, rds, tipc | Network protocol modules to blacklist |
| `hardening_modules_blacklist_extra` | `[]` | Additional modules to blacklist |
| `hardening_modules_blacklist_exclude` | `[]` | Modules to exclude from blacklisting |

### Filesystem Mounts

| Variable                   | Default                                     | Description                 |
| -------------------------- | ------------------------------------------- | --------------------------- |
| `hardening_fs_mounts`      | `/tmp`, `/dev/shm` with nodev,nosuid,noexec | Mount entries to harden     |
| `hardening_fs_vartmp_bind` | `true`                                      | Bind-mount /var/tmp to /tmp |

## Example Playbook

```yaml
- name: Apply system hardening
  ansible.builtin.include_role:
    name: marcstraube.common.hardening
  tags: [hardening]
  when: hardening_enabled | default(true) | bool
```

### Override for specific hosts

```yaml
# Allow squashfs (needed for Snap/AppImage) on a desktop
hardening_modules_blacklist_exclude:
  - squashfs

# Re-enable io_uring for PostgreSQL server
hardening_kernel_io_uring_disabled: 0

# Skip filesystem hardening on a system with non-standard mount layout
hardening_fs_enabled: false
```

## Tags

| Tag                    | Scope                  |
| ---------------------- | ---------------------- |
| `hardening`            | All hardening tasks    |
| `hardening:kernel`     | Sysctl configuration   |
| `hardening:modules`    | Module blacklisting    |
| `hardening:filesystem` | Mount option hardening |

## Testing

```bash
cd roles/hardening
molecule test
```

Driver: `vagrant` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

- `kernel.kexec_load_disabled` is a one-way toggle — once set to 1, it cannot be
  reverted without a reboot. If using kdump, ensure the crash kernel is loaded
  before this sysctl is applied.
- `kernel.perf_event_paranoid: 3` is a Debian/Arch kernel patch, not upstream.
  On RHEL/Rocky kernels, the maximum effective value is 2.
- `kernel.io_uring_disabled` requires kernel 6.6+ (backported to RHEL 9.4+).
  The template conditionally includes it based on the running kernel version.
- Module blacklisting takes effect after reboot or manual `modprobe -r`.
  Already-loaded modules are not unloaded by this role.

## License

MIT

## Author

Marc Straube
