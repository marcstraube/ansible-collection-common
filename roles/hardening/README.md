# marcstraube.common.hardening

Kernel hardening (sysctl), kernel module blacklisting, and filesystem mount
hardening.

## Description

This role handles three hardening areas:

- **Kernel sysctl** -- Security-focused kernel parameters via `/etc/sysctl.d/`
- **Module blacklisting** -- Disable unused filesystem and network kernel modules via `/etc/modprobe.d/`
- **Filesystem mounts** -- Harden mount options for `/tmp`, `/dev/shm`, `/var/tmp`

**Not in scope** (handled by other roles):

| Area                                         | Role                                         |
|----------------------------------------------|----------------------------------------------|
| Network sysctl (ip_forward, rp_filter, etc.) | `marcstraube.common.sysctl`                  |
| Audit daemon and rules                       | `marcstraube.common.auditd`                  |
| AppArmor                                     | `marcstraube.common.apparmor`                |
| Firejail sandboxing                          | `marcstraube.common.firejail`                |
| PAM hardening                                | `marcstraube.common.pam_hardening`           |
| GRUB kernel command line                     | `marcstraube.common.base` (bootloader tasks) |

## Requirements

- `ansible.posix` collection (for `ansible.posix.mount`)

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

| Variable                    | Default | Description                       |
|-----------------------------|---------|-----------------------------------|
| `hardening_enabled`         | `true`  | Enable the hardening role         |
| `hardening_kernel_enabled`  | `true`  | Enable kernel sysctl hardening    |
| `hardening_modules_enabled` | `true`  | Enable kernel module blacklisting |
| `hardening_fs_enabled`      | `true`  | Enable filesystem mount hardening |

### Kernel Sysctl

| Variable                                       | Default  | Description                                              |
|------------------------------------------------|----------|----------------------------------------------------------|
| `hardening_kernel_settings`                    | *(dict)* | Sysctl key-value pairs (see defaults)                    |
| `hardening_kernel_perf_event_paranoid_enabled` | `true`   | Deploy perf_event_paranoid (3 on Arch/Debian, 2 on RHEL) |
| `hardening_kernel_io_uring_disabled`           | `2`      | io_uring restriction (0/1/2). Kernel >= 6.6 only         |

### Module Blacklisting

| Variable                              | Default          | Description                     |
|---------------------------------------|------------------|---------------------------------|
| `hardening_modules_blacklist_fs`      | FS modules list  | Filesystem modules to blacklist |
| `hardening_modules_blacklist_net`     | Net modules list | Network modules to blacklist    |
| `hardening_modules_blacklist_extra`   | `[]`             | Additional modules              |
| `hardening_modules_blacklist_exclude` | `[]`             | Modules to exclude              |

### Filesystem Mounts

| Variable                           | Default            | Description                                       |
|------------------------------------|--------------------|---------------------------------------------------|
| `hardening_fs_mounts`              | `/tmp`, `/dev/shm` | Mount entries to harden                           |
| `hardening_fs_vartmp_bind_enabled` | `true`             | Bind /var/tmp to /tmp                             |
| `hardening_fs_tmp_force_tmpfs`     | `false`            | Force /tmp → tmpfs even when /tmp is not tmpfs    |

The `/tmp` entry in `hardening_fs_mounts` is only applied when the current
`/tmp` is already a tmpfs (e.g. systemd `tmp.mount` on Arch). On systems
where `/tmp` lives on the root filesystem, on a BTRFS subvolume, or on a
dedicated non-tmpfs partition, the entry is skipped to avoid masking
existing data or breaking services that hold open files there. Set
`hardening_fs_tmp_force_tmpfs: true` to opt in — appropriate for fresh
installs where `/tmp` is known to be empty.

### /proc hidepid

| Variable                                 | Default                    | Description                                |
|------------------------------------------|----------------------------|--------------------------------------------|
| `hardening_proc_hidepid_enabled`         | `false`                    | Mount /proc with hidepid (opt-in)          |
| `hardening_proc_hidepid_mode`            | `2`                        | `2` = invisible, `1` = listed not readable |
| `hardening_proc_hidepid_group`           | `'proc'`                   | Exemption group with full /proc visibility |
| `hardening_proc_hidepid_exempt_services` | `polkit`                   | Services granted the exemption via drop-in |

Hiding foreign process information keeps short-lived secrets that tools
accept as command-line arguments out of reach of unprivileged users.
Non-root services that must see foreign processes break under hidepid —
the role grants the exemption group through `SupplementaryGroups`
drop-ins and restarts the listed services before the remount. Add
further non-root services (e.g. monitoring agents that scan processes)
to `hardening_proc_hidepid_exempt_services` as needed. Root services
such as `systemd-logind` do not belong in the list: root bypasses
hidepid, and restarting logind under an active graphical session severs
the compositor's session devices. Drop-ins of services removed from the
list are pruned. Disabling the toggle rolls the drop-ins, the fstab
entry and the mount options back.

> **Warning — server hardening only.** hidepid is incompatible with
> desktop polkit authentication agents (KDE, XFCE, hyprpolkitagent and
> other PolkitQt/GLib agents): the agent process runs as the desktop
> user, fails to determine its own session under any hidepid level and
> cannot register — all polkit authentication dialogs stop working.
> This is a long-standing upstream limitation, not a configuration
> issue. Enable the toggle on servers without a graphical session.
> Adding a desktop user to the exemption group restores the agent but
> defeats the purpose, as that user's processes regain full /proc
> visibility.

## Tags

| Tag                    | Scope                  |
|------------------------|------------------------|
| `hardening`            | All hardening tasks    |
| `hardening:kernel`     | Sysctl configuration   |
| `hardening:modules`    | Module blacklisting    |
| `hardening:filesystem` | Mount option hardening |
| `hardening:proc`       | /proc hidepid mounting |

## Example Playbook

```yaml
- name: Include hardening role
  ansible.builtin.include_role:
    name: marcstraube.common.hardening
  tags:
    - hardening
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

## Testing

```bash
cd roles/hardening
molecule test
```

Driver: `vagrant` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

- `kernel.kexec_load_disabled` is a one-way toggle -- once set to 1, it cannot be
  reverted without a reboot. If using kdump, ensure the crash kernel is loaded
  before this sysctl is applied.
- `kernel.perf_event_paranoid: 3` is a Debian/Arch kernel patch, not upstream.
  On RHEL/Rocky kernels, the maximum effective value is 2.
- `kernel.io_uring_disabled` requires kernel 6.6+ (backported to RHEL 9.4+).
  The template conditionally includes it based on the running kernel version.
- Module blacklisting takes effect after reboot or manual `modprobe -r`.
  Already-loaded modules are not unloaded by this role.

### GRUB Hardening Parameters

These kernel command line parameters complement this role's sysctl settings.
Configure them via the base role's bootloader variables:

| Parameter                  | Effect                                                        |
|----------------------------|---------------------------------------------------------------|
| `lockdown=integrity`       | Prevent modification of running kernel (modules, /dev/mem)    |
| `lockdown=confidentiality` | Additionally block reading kernel memory (breaks hibernation) |
| `init_on_alloc=1`          | Zero memory at allocation (~1% perf cost)                     |
| `init_on_free=1`           | Zero memory at free (~5% perf cost, skip on DB servers)       |
| `vsyscall=none`            | Disable legacy vsyscall (safe on modern glibc 2.14+)          |
| `debugfs=off`              | Prevent mounting debugfs                                      |
| `page_alloc.shuffle=1`     | Randomize page allocator free lists                           |
| `iommu=force`              | Force IOMMU isolation (5-15% I/O cost, skip in VMs)           |

## References

- [CIS Benchmark](https://www.cisecurity.org/benchmark/distribution_independent_linux) — hardening guidelines
- [DISA STIGs](https://www.stig.dod.mil/) — Security Technical Implementation Guides

## License

MIT

## Author

Marc Straube
