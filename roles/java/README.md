# marcstraube.common.java

Install an OpenJDK release and make it the default Java.

## Requirements

- ansible-core >= 2.19

## Supported Platforms

| Platform                   | Notes            |
|----------------------------|------------------|
| Arch Linux                 | 17, 21, 25       |
| Debian Trixie              | 21, 25           |
| EL 9 (Rocky, Alma, RHEL)   | 17, 21, 25       |
| EL 10 (Rocky, Alma, RHEL)  | 17, 21, 25       |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable       | Default | Description          |
|----------------|---------|----------------------|
| `java_enabled` | `false` | Enable the java role |

### Installation

| Variable       | Default | Description                        |
|----------------|---------|------------------------------------|
| `java_version` | `25`    | OpenJDK release to install (LTS)   |

The role installs the JDK package of the chosen release: `jdk<version>-openjdk`
on Arch Linux, `openjdk-<version>-jdk` on Debian, `java-<version>-openjdk-devel`
on EL. The release has to be one the platform packages; the role fails
otherwise. Arch Linux keeps several JDKs side by side, so the role selects the
chosen release with `archlinux-java set`. Debian and EL select the default
through alternatives when the package is installed.

## Tags

| Tag            | Description |
|----------------|-------------|
| `java`         | All tasks   |
| `java:install` | JDK install |

## Example Playbook

```yaml
- hosts: workstations
  roles:
    - role: marcstraube.common.java
      vars:
        java_enabled: true
        java_version: '21'
```

## Testing

```bash
cd roles/java
molecule test
```

The scenario installs OpenJDK 25 on all four platforms and checks that `java`
and `javac` report it.

## References

- [OpenJDK](https://openjdk.org/)
- [Arch Linux Java wiki](https://wiki.archlinux.org/title/Java)

## License

MIT

## Author

Marc Straube
