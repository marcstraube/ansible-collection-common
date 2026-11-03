# marcstraube.common.vendor_packages

Build and install packages from vendor files that cannot be downloaded
unattended.

Some vendors require a login and an EULA acceptance before the download
(Unreal Engine for Linux, for example). Roles cannot fetch such files
themselves, so the user downloads them on the Ansible controller and places
them in a directory; this role stages the file on the target and installs the
package. Other roles include one of its entry points with `include_role` and
`tasks_from` instead of each implementing its own install path. The role has
no standalone run.

Entry points:

| Entry point | Platform   | Description                                                |
|-------------|------------|------------------------------------------------------------|
| `aur.yml`   | Arch Linux | AUR package whose PKGBUILD has a `manual://` source        |

## Requirements

- ansible-core >= 2.19
- The `kewlfft.aur` collection
- The AUR builder user set up by `marcstraube.common.package_management`
  (`aur_builder_user`); the role builds as that user and reads
  `aur_builder_home` and `makepkg_tmpdir` from the same role when they are set

## Supported Platforms

| Platform   | Notes                 |
|------------|-----------------------|
| Arch Linux | `aur.yml` entry point |

Other distributions in the same os_family (EndeavourOS, Manjaro) should work
but are not actively tested.

## Role Variables

The role has no `vendor_packages_enabled` toggle: it never runs on its own,
and the calling roles keep their own toggles for the packages they install.

### Vendor Files

| Variable                    | Default                            | Description                                      |
|-----------------------------|------------------------------------|--------------------------------------------------|
| `vendor_packages_dir`       | `{{ inventory_dir }}/files/vendor` | Controller directory with the downloaded files   |
| `vendor_packages_build_dir` | `''`                               | Build directory on the target (empty: see below) |

`vendor_packages_dir` is a directory on the Ansible controller: any absolute
path works, for example a mounted network share, and the default is
`files/vendor/` next to the inventory:

```text
inventories/<name>/
├── hosts.yml
├── group_vars/
└── files/
    └── vendor/
        └── Linux_Unreal_Engine_5.6.1.zip
```

Vendor files are large and often under a license that forbids
redistribution. When the inventory is kept in git, add `files/vendor/` to its
`.gitignore`.

`vendor_packages_build_dir` holds the clone of every package repository and,
in `sources/`, the staged vendor file while the package builds. Empty, the
role uses `vendor_packages` below the home of the AUR builder user
(`aur_builder_home` from `marcstraube.common.package_management`,
`/var/cache/aur_builder` by default). The directory needs free space for the
staged file, the extracted sources and the package, about twice the vendor
file plus the package. The build itself runs below
`makepkg_tmpdir` when `marcstraube.common.package_management` sets it, and
below the build directory otherwise; never below `/tmp`, which is often a
tmpfs in RAM. After a successful build the role removes the staged file; the
clone stays for the version check of the next run.

### Caller Variables

The calling role passes the package values as `vars` of the `include_role`
task. They describe one package and have no defaults.

| Variable                        | Required | Description                                                       |
|---------------------------------|----------|-------------------------------------------------------------------|
| `vendor_packages_aur_name`      | yes      | Name of the AUR package (the repository `<name>.git`)             |
| `vendor_packages_file_pattern`  | yes      | File name of the vendor file; `{pkgver}` stands for the version   |
| `vendor_packages_download_url`  | no       | Where the user downloads the file; printed in the notice          |

`vendor_packages_file_pattern` is the file name as the PKGBUILD expects it
next to itself. The role replaces `{pkgver}` with the `pkgver` of the
PKGBUILD, so a new release of the package asks for a new file; a file of an
older version has another name and counts as missing.

### Behavior of aur.yml

For every call the role:

1. queries the package in the configured pacman repositories (`pacman -Si`).
   When a repository carries it, the role installs it from there and stops;
   see [Prebuilt Packages](#prebuilt-packages)
2. clones the package repository from the AUR into the build directory as the
   AUR builder user, or updates an existing clone
3. reads `pkgver`, `pkgrel` and `epoch` from the `.SRCINFO` of the clone and
   derives the expected file name from `vendor_packages_file_pattern`
4. compares the version with the installed package (`vercmp`); when the
   package is installed in that version or a newer one, nothing else happens
5. checks the file in `vendor_packages_dir` on the controller
6. with the file present: stages it in the `sources/` directory of the build
   directory (`ansible.builtin.copy`, streamed through the connection, so a
   file of several GB needs no extra memory) and builds and installs the
   package with `kewlfft.aur.aur` and `local_pkgbuild` as the AUR builder
   user. makepkg reads the file from there (`SRCDEST`) and verifies it
   against the checksums of the PKGBUILD. Afterwards the staged file is
   removed
7. with the file missing: prints a notice with the expected file name, the
   version and `vendor_packages_download_url`, and skips the package without
   failing the play

The role is install-only: it never removes a package and never downgrades
one. A build that fails (wrong checksum, build error) fails the play and
leaves the staged file in place, so the next run does not transfer it again.

The notice looks like this:

```text
Vendor file for unreal-engine-bin 5.6.1-1 not found on the Ansible controller:
/home/user/inventories/prod/files/vendor/Linux_Unreal_Engine_5.6.1.zip.
Download Linux_Unreal_Engine_5.6.1.zip from https://www.unrealengine.com/linux
and place it in /home/user/inventories/prod/files/vendor; the package is
skipped on this run.
```

### Check Mode

The clone and the version query run in check mode too, because they decide
what would change. The transfer and the build are reported as changes and
not executed.

## Tags

The role defines no tags of its own. The `include_role` task of the calling
role carries the tags; use `apply` to pass them on to the included tasks:

```yaml
- name: Game Engines | Install Unreal Engine from the vendor file
  ansible.builtin.include_role:
    name: marcstraube.common.vendor_packages
    tasks_from: aur.yml
    apply:
      tags:
        - game-engines
        - game-engines:unreal
  vars:
    vendor_packages_aur_name: 'unreal-engine-bin'
    vendor_packages_file_pattern: 'Linux_Unreal_Engine_{pkgver}.zip'
    vendor_packages_download_url: 'https://www.unrealengine.com/linux'
  tags:
    - game-engines
    - game-engines:unreal
```

## Example Playbook

The role is meant for other roles. A calling role gates the include with its
own toggle and passes the package values:

```yaml
- name: Game Engines | Install Unreal Engine from the vendor file
  ansible.builtin.include_role:
    name: marcstraube.common.vendor_packages
    tasks_from: aur.yml
  vars:
    vendor_packages_aur_name: 'unreal-engine-bin'
    vendor_packages_file_pattern: 'Linux_Unreal_Engine_{pkgver}.zip'
    vendor_packages_download_url: 'https://www.unrealengine.com/linux'
  when:
    - ansible_facts['os_family'] == 'Archlinux'
    - game_engines_unreal_enabled | bool
```

The same call works from a playbook:

```yaml
- hosts: workstations
  become: true
  tasks:
    - name: Install Unreal Engine from the vendor file
      ansible.builtin.include_role:
        name: marcstraube.common.vendor_packages
        tasks_from: aur.yml
      vars:
        vendor_packages_dir: '{{ inventory_dir }}/files/vendor'
        vendor_packages_aur_name: 'unreal-engine-bin'
        vendor_packages_file_pattern: 'Linux_Unreal_Engine_{pkgver}.zip'
        vendor_packages_download_url: 'https://www.unrealengine.com/linux'
```

## Testing

```bash
cd roles/vendor_packages
molecule test
```

The scenario runs on Arch Linux only. `prepare.yml` creates the AUR builder
user and serves two fixture packages (`molecule/default/files/aur`) from
local bare repositories inside the container; `converge.yml` points the role
at them instead of the AUR. The vendor directory of the scenario
(`molecule/default/files/vendor`) holds a small dummy archive for
`vendor-dummy` and an archive of an older version for `vendor-dummy-absent`:

- `vendor-dummy`: file present, built and installed; the second run changes
  nothing
- `vendor-dummy-absent` with the file of another version: treated as
  missing, notice, play goes on
- `vendor-dummy-absent` without a vendor directory: missing, notice, play
  goes on
- `vendor-repo-dummy`: provided by a local pacman repository
  (`molecule/default/files/repo`, served from a `file://` URL), installed
  from there without a clone or a vendor file

`verify.yml` checks the installed package and its content, that
`vendor-dummy-absent` is not installed, and that the build directory holds
the clones and an empty `sources/` directory only.

## Notes

### Prebuilt Packages

When a pacman repository, for example a private one, already provides the
prebuilt package, there is nothing to build on the host: add the repository
through `marcstraube.common.package_management` and keep the toggle of the
calling role enabled. The role finds the package in the repository, installs
it from there and skips the vendor path, so the calling role still runs its
own setup tasks. Updates of such a package arrive with the system upgrade,
not through the role. The repository is queried for `vendor_packages_aur_name`,
so for a split package the repository has to carry a package of that name.

### Split Packages

The version check queries the first `pkgname` of the `.SRCINFO`. Split
packages share one version, so the first package decides for all of them.

### Vendor Files on the Controller

A vendor file is read on the controller only when the package needs a
build: `ansible.builtin.copy` computes a checksum of the file before the
transfer, which takes a moment for a file of several GB. On runs where the
installed package already matches, the file is not touched.

## References

- [makepkg: sources that must be downloaded by hand](https://wiki.archlinux.org/title/PKGBUILD#source)
- [kewlfft.aur collection](https://github.com/kewlfft/ansible-aur)
- [AUR submission guidelines](https://wiki.archlinux.org/title/AUR_submission_guidelines)

## License

MIT

## Author

Marc Straube
