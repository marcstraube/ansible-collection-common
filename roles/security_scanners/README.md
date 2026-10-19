# marcstraube.common.security_scanners

Install language-independent supply-chain and secret scanners: Socket CLI,
trivy, gitleaks, osv-scanner and syft.

Scanners bound to one language live in that language's role: bandit in
`python`, cargo-audit in `rust`, govulncheck in `go`.

## Requirements

- ansible-core >= 2.19
- Collections: `community.general`, `kewlfft.aur` (Socket CLI on Arch Linux)
- On Arch Linux: an `aur_builder` user for the AUR package (set up by
  `package_management`)
- On Debian and EL: npm with Node.js 20 or newer for the Socket CLI (the
  `nodejs` role; the default `nodejs` stream of EL 9 is 16 and too old)
- On EL 10: EPEL for trivy (enabled by `package_management`)

## Supported Platforms

| Platform                   | Notes                                      |
|----------------------------|--------------------------------------------|
| Arch Linux                 | all scanners                               |
| Debian Trixie              | Socket CLI, gitleaks                       |
| EL 9 (Rocky, Alma, RHEL)   | Socket CLI                                 |
| EL 10 (Rocky, Alma, RHEL)  | Socket CLI, trivy                          |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable                    | Default | Description                       |
|-----------------------------|---------|-----------------------------------|
| `security_scanners_enabled` | `false` | Enable the security_scanners role |

### Scanners

| Variable                                | Default | Description                                                      |
|-----------------------------------------|---------|------------------------------------------------------------------|
| `security_scanners_socket_enabled`      | `false` | Socket CLI, dependency supply-chain scanner (npm, PyPI, Go, ...) |
| `security_scanners_trivy_enabled`       | `false` | trivy, vulnerability and misconfiguration scanner                |
| `security_scanners_gitleaks_enabled`    | `false` | gitleaks, secret scanner for git repositories                    |
| `security_scanners_osv_scanner_enabled` | `false` | osv-scanner, lockfile scanner against the OSV database           |
| `security_scanners_syft_enabled`        | `false` | syft, SBOM generator                                             |

The Socket CLI is distributed through npm; Arch Linux has it as the AUR
package `socket-cli`, Debian and EL install it as a global npm package. The
other scanners come from distribution packages where the platform has them
(see Supported Platforms); an enabled scanner without a package is skipped.
The role installs and never removes.

## Tags

| Tag                         | Description |
|-----------------------------|-------------|
| `security-scanners`         | All tasks   |
| `security-scanners:install` | Scanners    |

## Example Playbook

```yaml
- hosts: workstations
  roles:
    - role: marcstraube.common.nodejs
    - role: marcstraube.common.security_scanners
      vars:
        security_scanners_enabled: true
        security_scanners_socket_enabled: true
        security_scanners_gitleaks_enabled: true
        security_scanners_trivy_enabled: true
```

## Testing

```bash
cd roles/security_scanners
molecule test
```

The scenario enables every scanner on all four platforms and checks that the
packaged ones and the Socket CLI run.

## References

- [Socket CLI](https://github.com/SocketDev/socket-cli)
- [trivy](https://trivy.dev/), [gitleaks](https://github.com/gitleaks/gitleaks),
  [osv-scanner](https://google.github.io/osv-scanner/), [syft](https://github.com/anchore/syft)

## License

MIT

## Author

Marc Straube
