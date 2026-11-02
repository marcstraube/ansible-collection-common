# marcstraube.common.vagrant

Install Vagrant, set the default provider and install Vagrant plugins per
user.

## Requirements

- ansible-core >= 2.19
- On Arch Linux: Vagrant is installed from the AUR, which needs the AUR
  builder user set up by the `package_management` role and the
  `kewlfft.aur` collection
- On Debian and EL: Vagrant comes from the HashiCorp repository, which the
  `package_management` role adds (see [HashiCorp Repository](#hashicorp-repository))
- A provider from its own role: `marcstraube.common.virtualbox`
  (`virtualbox_enabled`) or `marcstraube.common.libvirt` (`libvirt_enabled`).
  This role installs Vagrant only, never a provider

## Supported Platforms

| Platform                   | Notes                          |
|----------------------------|--------------------------------|
| Arch Linux                 | AUR package `vagrant`          |
| Debian Trixie              | HashiCorp apt repository       |
| EL 9 (Rocky, Alma, RHEL)   | HashiCorp dnf repository       |
| EL 10 (Rocky, Alma, RHEL)  | HashiCorp dnf repository       |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable          | Default | Description             |
|-------------------|---------|-------------------------|
| `vagrant_enabled` | `false` | Enable the vagrant role |

### Provider

| Variable                         | Default | Description                                                    |
|----------------------------------|---------|----------------------------------------------------------------|
| `vagrant_default_provider`       | `''`    | Default provider (`virtualbox`, `libvirt`, or empty to unset)  |
| `vagrant_libvirt_plugin_enabled` | `false` | Install the `vagrant-libvirt` plugin for the `vagrant_users`   |

With a provider set, the role exports it as `VAGRANT_DEFAULT_PROVIDER` in
`/etc/profile.d/vagrant.sh`, which login shells read. An empty value removes
the file, and Vagrant uses the first usable provider. A `Vagrantfile` or
`vagrant up --provider` still overrides the default.

Vagrant supports libvirt through the `vagrant-libvirt` plugin. Independent of
the default provider, `vagrant_libvirt_plugin_enabled`:

- installs the build dependencies of the `vagrant-libvirt` plugin (compiler,
  `make`, `pkg-config` and the libvirt development headers)
- adds `vagrant-libvirt` to the plugins of every entry in `vagrant_users`

On EL the libvirt development headers (`libvirt-devel`) come from CRB, which
the `package_management` role enables with `dnf_powertools_enabled: true`.

### Plugins

| Variable        | Default | Description                                  |
|-----------------|---------|----------------------------------------------|
| `vagrant_users` | `[]`    | Users and the plugins installed for each one |

Each entry has a `username` and an optional list of `plugins`. Vagrant keeps
plugins in the user's home (`~/.vagrant.d`), so the role runs
`vagrant plugin install` as that user. A plugin that `vagrant plugin list`
already shows is skipped; the role neither updates nor removes plugins.

Vagrant installs plugins for one Ruby version. After a Ruby version change
(on Arch Linux with a Ruby update, on Debian and EL with a Vagrant release that
ships a newer Ruby) every Vagrant command fails with "The plugins failed to
initialize correctly". When `vagrant plugin list` fails that way, the role runs
`vagrant plugin expunge --reinstall --force` for that user, which reinstalls
all plugins of the user for the current Ruby.

```yaml
vagrant_users:
  - username: 'johndoe'
    plugins:
      - 'vagrant-hostmanager'
  - username: 'developer'
```

### HashiCorp Repository

Debian and EL install Vagrant from the HashiCorp repository. The role does not
add it; the `package_management` role does, through the inventory. The
`vagrant` package in the Debian archive is older and supports VirtualBox only
up to 7.0; without the HashiCorp repository Debian installs that package.

Debian (`apt_custom_repos`):

```yaml
apt_custom_repos:
  - name: 'hashicorp'
    uris: 'https://apt.releases.hashicorp.com'
    suites: 'trixie'
    components: 'main'
    gpg_key: 'https://apt.releases.hashicorp.com/gpg'
```

EL (`dnf_custom_repos`), plus CRB for the libvirt provider:

```yaml
dnf_custom_repos:
  - name: 'hashicorp'
    description: 'HashiCorp Stable - $basearch'
    baseurl: 'https://rpm.releases.hashicorp.com/RHEL/$releasever/$basearch/stable'
    gpgkey: 'https://rpm.releases.hashicorp.com/gpg'
    gpgcheck: true

# Only with vagrant_libvirt_plugin_enabled: true
dnf_powertools_enabled: true
```

Vagrant is not pinned; it updates with the system through the repository (or
the AUR helper on Arch Linux).

## Tags

| Tag                 | Description                       |
|---------------------|-----------------------------------|
| `vagrant`           | All tasks                         |
| `vagrant:install`   | Vagrant and build dependencies    |
| `vagrant:configure` | Default provider in profile.d     |
| `vagrant:plugins`   | Per-user plugins                  |

## Example Playbook

VirtualBox as provider:

```yaml
- hosts: workstations
  roles:
    - role: marcstraube.common.package_management
      vars:
        apt_custom_repos:
          - name: 'hashicorp'
            uris: 'https://apt.releases.hashicorp.com'
            suites: 'trixie'
            components: 'main'
            gpg_key: 'https://apt.releases.hashicorp.com/gpg'
        dnf_custom_repos:
          - name: 'hashicorp'
            description: 'HashiCorp Stable - $basearch'
            baseurl: 'https://rpm.releases.hashicorp.com/RHEL/$releasever/$basearch/stable'
            gpgkey: 'https://rpm.releases.hashicorp.com/gpg'
            gpgcheck: true
    - role: marcstraube.common.virtualbox
      vars:
        virtualbox_enabled: true
    - role: marcstraube.common.vagrant
      vars:
        vagrant_enabled: true
        vagrant_users:
          - username: 'johndoe'
            plugins:
              - 'vagrant-hostmanager'
```

libvirt as provider (the repository entries as above, plus CRB on EL):

```yaml
- hosts: workstations
  roles:
    - role: marcstraube.common.libvirt
      vars:
        libvirt_enabled: true
    - role: marcstraube.common.vagrant
      vars:
        vagrant_enabled: true
        vagrant_default_provider: 'libvirt'
        vagrant_libvirt_plugin_enabled: true
        vagrant_users:
          - username: 'johndoe'
```

## Testing

```bash
cd roles/vagrant
molecule test
```

The scenario adds the HashiCorp repository in `prepare.yml` (Debian, EL),
installs Vagrant on all four platforms and checks the package, the profile.d
file and the plugin list of the test users. `side_effect.yml` removes the
installed plugin gems of one user, as a Ruby version change leaves them, and
runs the role again, which has to reinstall the plugins. Starting a box is not
part of the scenario; no provider runs inside the test container.

## References

- [Vagrant documentation](https://developer.hashicorp.com/vagrant/docs)
- [Vagrant installation and HashiCorp repositories](https://developer.hashicorp.com/vagrant/install)
- [Vagrant plugins](https://developer.hashicorp.com/vagrant/docs/plugins/usage)
- [vagrant-libvirt installation](https://vagrant-libvirt.github.io/vagrant-libvirt/installation.html)

## License

MIT

## Author

Marc Straube
