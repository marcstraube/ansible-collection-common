# marcstraube.common.storage

Set up additional disks with LUKS2, btrfs, LVM or ext4, and install tools for
common filesystems.

## Description

The role sets up data disks on an installed system, end to end:

1. **Partition:** GPT with a single partition spanning the disk, labeled
   `storage-<name>`. The label is how the role finds the volume again.
2. **LUKS2** (optional): the container is created with a passphrase, which
   stays as the recovery key. The role then adds a random keyfile in a keyslot
   of its own (see [Encryption](#encryption)).
3. **Layout:**
   - `btrfs`: one btrfs filesystem with subvolumes, each mounted on its own
     path, optionally with a snapper config
   - `lvm`: an LVM volume group with one ext4 filesystem per logical volume
   - `ext4`: a single ext4 filesystem
4. **crypttab and fstab** entries by UUID; mount points are created, mounted,
   and get the configured owner, group and mode.

Existing volumes work as well: with `format: false` the role opens and mounts
what is there, adds the keyfile slot to an existing LUKS container, and
creates missing subvolumes or logical volumes.

The role also installs tools for filesystems it does not set up itself: NTFS
(dual boot, external disks), exFAT and LVM. For VeraCrypt volumes, see
`marcstraube.desktop.security_apps`.

An installer role can run the role from a live medium against the new system
mounted below a chroot path, so a volume (for example `/home`) is in place
before the first boot (see [Installer Mode](#installer-mode)).

## Requirements

- ansible-core >= 2.19
- `community.general` collection (`parted`, `filesystem`, `lvg`, `lvol`,
  `crypttab`)
- `community.crypto` collection (`luks_device`)
- `ansible.posix` collection (`mount`)
- EL: EPEL enabled for `ntfs-3g`

## Supported Platforms

| Platform                  | Notes                                         |
| ------------------------- | --------------------------------------------- |
| Arch Linux                |                                               |
| Debian Trixie             |                                               |
| EL 9 (Rocky, Alma, RHEL)  | No `btrfs` layout (the kernel lacks btrfs)    |
| EL 10 (Rocky, Alma, RHEL) | No `btrfs` layout (the kernel lacks btrfs)    |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable          | Default | Description             |
| ----------------- | ------- | ----------------------- |
| `storage_enabled` | `true`  | Enable the storage role |

### Volumes

| Variable          | Default | Description          |
| ----------------- | ------- | -------------------- |
| `storage_volumes` | `[]`    | Volumes to set up    |

Each volume:

| Key               | Required      | Description                                                                    |
| ----------------- | ------------- | ------------------------------------------------------------------------------ |
| `name`            | yes           | `[a-z0-9_-]+`; derives the partition label, mapper name and VG name            |
| `device`          | yes           | Disk (`/dev/disk/by-id/...`); with `format: false` also an existing partition  |
| `layout`          | yes           | `btrfs`, `lvm` or `ext4`                                                       |
| `format`          | no (`false`)  | Partition and format the device (see [Format Safety](#format-safety))          |
| `force`           | no (`false`)  | With `format`: wipe existing content                                           |
| `luks.enabled`    | no (`false`)  | Encrypt with LUKS2                                                             |
| `luks.passphrase` | with `format` | Recovery passphrase; also needed once to enroll the keyfile                    |
| `luks.name`       | no            | Mapper name (default `crypt_<name>`)                                           |
| `luks.options`    | no (`luks`)   | crypttab options, e.g. `luks,discard`                                          |
| `mount_options`   | no            | Mount options for all mount points of the volume                               |
| `subvolumes`      | `btrfs`       | List of subvolumes (below)                                                     |
| `vg_name`         | no            | Volume group name for `lvm` (default `vg_<name>`)                              |
| `logical_volumes` | `lvm`         | List of logical volumes (below)                                                |
| `path`            | `ext4`        | Mount path                                                                     |
| `owner`           | no            | Mount point owner for `ext4`                                                   |
| `group`           | no            | Mount point group for `ext4`                                                   |
| `mode`            | no            | Mount point mode for `ext4`                                                    |

btrfs subvolume (`subvolumes`):

| Key                | Required | Description                                                          |
| ------------------ | -------- | -------------------------------------------------------------------- |
| `name`             | yes      | Subvolume path below the top level, e.g. `@/data`                    |
| `path`             | yes      | Mount path                                                           |
| `owner`            | no       | Mount point owner                                                    |
| `group`            | no       | Mount point group                                                    |
| `mode`             | no       | Mount point mode                                                     |
| `mount_options`    | no       | Appended to the volume's `mount_options`                             |
| `snapper`          | no       | Create a snapper config for the subvolume                            |
| `snapper_config`   | no       | Config name (default: mount path without the leading `/`, `/` → `_`) |
| `snapper_settings` | no       | Merged over `storage_snapper_settings`                               |

LVM logical volume (`logical_volumes`):

| Key             | Required | Description                                |
| --------------- | -------- | ------------------------------------------ |
| `name`          | yes      | Logical volume name                        |
| `size`          | yes      | `lvcreate` size, e.g. `500g` or `100%FREE` |
| `path`          | yes      | Mount path                                 |
| `owner`         | no       | Mount point owner                          |
| `group`         | no       | Mount point group                          |
| `mode`          | no       | Mount point mode                           |
| `mount_options` | no       | Appended to the volume's `mount_options`   |

### Snapper

| Variable                   | Default                                  | Description                       |
| -------------------------- | ---------------------------------------- | --------------------------------- |
| `storage_snapper_settings` | hourly 5, daily 7, number limit 50       | Default snapper config settings   |

Keys are snapper config variables (`TIMELINE_CREATE`, `TIMELINE_LIMIT_DAILY`,
`NUMBER_LIMIT`, ...); see `defaults/main.yml` for the full default.

### Installer Mode

| Variable                 | Default | Description                                                  |
| ------------------------ | ------- | ------------------------------------------------------------ |
| `storage_installer_mode` | `false` | Operate on a new system mounted below `storage_chroot_path`  |
| `storage_chroot_path`    | `/mnt`  | Mount point of the new system in installer mode              |

In installer mode the role sets the fact `storage_installer_packages`: the
packages the new system needs for its volumes (see [Installer Mode](#installer-mode)).

### Filesystem Tools

| Variable                | Default | Description                                                         |
| ----------------------- | ------- | ------------------------------------------------------------------- |
| `storage_ntfs_enabled`  | `false` | `ntfs-3g`: NTFS tools; mounting uses the `ntfs3` kernel driver      |
| `storage_exfat_enabled` | `false` | `exfatprogs`: exFAT tools (USB sticks, SD cards)                    |
| `storage_lvm_enabled`   | `false` | `lvm2` without an `lvm` volume, e.g. to inspect existing disks      |

The toggles only install. `cryptsetup`, `parted`, `btrfs-progs`, `lvm2`,
`e2fsprogs` and `snapper` are installed when a volume needs them.

### Examples

#### Encrypted btrfs data disk with snapper

```yaml
storage_volumes:
  - name: 'data'
    device: '/dev/disk/by-id/nvme-Example_SSD_1TB_SERIAL'
    format: true
    luks:
      enabled: true
      passphrase: '{{ vault_storage_data_passphrase }}'
      options: 'luks,discard'
    layout: 'btrfs'
    mount_options: 'compress-force=zstd:3,ssd,nodev,nosuid'
    subvolumes:
      - name: '@/data'
        path: '/data'
        owner: 'johndoe'
        group: 'johndoe'
        mode: '0750'
        snapper: true
```

Moving `/home` to this disk later is one more entry, e.g. `name: '@/home'`,
`path: '/home'`, after copying the data over.

#### Encrypted LVM with ext4 for backups

```yaml
storage_volumes:
  - name: 'backup'
    device: '/dev/disk/by-id/ata-Example_HDD_2TB_SERIAL'
    format: true
    luks:
      enabled: true
      passphrase: '{{ vault_storage_backup_passphrase }}'
    layout: 'lvm'
    mount_options: 'nodev,nosuid,noexec'
    logical_volumes:
      - name: 'restic'
        size: '1t'
        path: '/srv/backup/restic'
        owner: 'root'
        group: 'root'
        mode: '0700'
      - name: 'archive'
        size: '100%FREE'
        path: '/srv/backup/archive'
```

#### Existing LUKS volume

```yaml
storage_volumes:
  - name: 'old'
    device: '/dev/disk/by-id/ata-Example_HDD_1TB_SERIAL-part1'
    luks:
      enabled: true
      passphrase: '{{ vault_storage_old_passphrase }}'
    layout: 'ext4'
    path: '/srv/old'
```

## Tags

| Tag               | Scope                                   |
| ----------------- | --------------------------------------- |
| `storage`         | All role tasks                          |
| `storage:install` | Package installation                    |
| `storage:volumes` | Partitions, LUKS, filesystems, mounts   |
| `storage:service` | snapper timers                          |

## Example Playbook

```yaml
- name: Storage
  hosts: all
  become: true
  tasks:
    - name: Include storage role
      ansible.builtin.include_role:
        name: marcstraube.common.storage
      tags: [storage]
      when: storage_enabled | default(true) | bool
```

## Testing

```bash
cd roles/storage
MOLECULE_PLATFORM_NAME=storage-archlinux molecule test -p storage-archlinux
```

Driver: `vagrant` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

The scenario uses loop devices inside the VM: LUKS2 + btrfs + snapper (Arch,
Debian), LUKS2 + LVM + ext4, ext4 over an existing filesystem with `force`,
LUKS2 + ext4 in installer mode below a chroot directory, refusal of a
non-blank device without `force`, and rejection of btrfs on EL.

## Notes

### Encryption

- The passphrase is set when the container is created and stays in its
  keyslot as the recovery key.
- The role generates a random 4096-byte keyfile
  `/etc/cryptsetup-keys.d/<mapper name>.key` (root, `0400`) and adds it in a
  keyslot of its own; the passphrase is never written to disk.
- crypttab unlocks the volume with that keyfile at boot. The keyfile lives on
  the root filesystem, so it is only readable once that is unlocked; encrypt
  the root filesystem as well.
- Once the keyfile is enrolled, the role tests it on every run and needs the
  passphrase only to enroll a missing keyfile again.
- To revoke the keyfile, remove its keyslot
  (`cryptsetup luksRemoveKey --key-file <keyfile> <device>`) and delete the
  file; the passphrase keeps working.

### Format Safety

| `format` | `force` | Device state                       | Result                                     |
| -------- | ------- | ---------------------------------- | ------------------------------------------ |
| `false`  | –       | any                                | Never formats; opens and mounts content    |
| `true`   | `false` | blank                              | Partitions and formats                     |
| `true`   | `false` | filesystem, LUKS, LVM, partitions  | Fails and names what it found              |
| `true`   | `true`  | filesystem, LUKS, LVM, partitions  | Wipes all signatures, then formats         |
| `true`   | any     | already set up by this role        | Leaves it as it is                         |

The role also refuses to format a device that is mounted or has opened
children (LUKS, LVM, RAID). A volume set up by this role is recognized by its
partition label, so `format: true` can stay in the inventory.

### Installer Mode

With `storage_installer_mode: true` the role runs on a live medium (Arch ISO,
for example) against the new system mounted at `storage_chroot_path`:

- Partitioning, LUKS and filesystems are set up on the live system as usual.
- The keyfile goes to `<chroot>/etc/cryptsetup-keys.d/`, the crypttab entry to
  `<chroot>/etc/crypttab`; both refer to paths as the new system sees them.
- Mount points are mounted below the chroot path. The role does not write
  fstab there: the installer's fstab generation (`genfstab` on Arch) records
  the mounts along with the system's own.
- Packages are not installed; the live medium ships the tools. The role sets
  the fact `storage_installer_packages` with the packages the new system
  needs, and the installer installs them into the new system.
- snapper configs, snapper timers and the systemd reload are skipped. The
  installer runs the role again on the installed system after the first boot;
  that run finds the volume in place, applies the mount point ownership and
  adds snapper and the timers. It also rewrites the fstab entry in the role's
  own format once.
- The OS-specific variables come from the live medium, so it must share the
  os_family of the new system.
- `owner`, `group` and `mode` of mount points are not applied; the run on the
  installed system sets them once its users and groups exist.

An installer typically runs the role once in this mode after the base system
is installed into the chroot and before it generates fstab, and once more on
the installed system.

### Other Notes

- Check mode works against existing volumes. For a volume that is not set up
  yet, it reports the partitioning and skips the rest, since the partition
  does not exist.
- Missing parent path components of a subvolume (`@` in `@/data`) are created
  as plain directories.
- The snapper timers (`snapper-timeline.timer`, `snapper-cleanup.timer`) serve
  all snapper configs on the host. The role enables them when it sets up a
  snapper config and leaves them alone otherwise.
- Logical volumes grow when their `size` is raised (`resizefs`); they never
  shrink.

## References

- [crypttab(5)](https://man.archlinux.org/man/crypttab.5) — keyfile lookup in `/etc/cryptsetup-keys.d/`
- [ArchWiki: dm-crypt/Encrypting a non-root file system](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_a_non-root_file_system)
- [ArchWiki: Snapper](https://wiki.archlinux.org/title/Snapper)

## License

MIT

## Author

Marc Straube
