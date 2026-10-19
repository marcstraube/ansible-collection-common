# marcstraube.common.go

Install the Go toolchain with govulncheck.

## Requirements

- ansible-core >= 2.19

## Supported Platforms

| Platform                   | Notes |
|----------------------------|-------|
| Arch Linux                 |       |
| Debian Trixie              |       |
| EL 9 (Rocky, Alma, RHEL)   |       |
| EL 10 (Rocky, Alma, RHEL)  |       |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable     | Default | Description        |
|--------------|---------|--------------------|
| `go_enabled` | `false` | Enable the go role |

The toolchain comes from the distribution: `go` on Arch Linux, `golang` on
Debian and EL.

### Security Tooling

| Variable                 | Default | Description                                            |
|--------------------------|---------|--------------------------------------------------------|
| `go_govulncheck_enabled` | `false` | Install govulncheck (Go vulnerability database checks) |

govulncheck is packaged on Arch Linux and Debian. EL gets a notice and can
install it with `go install golang.org/x/vuln/cmd/govulncheck@latest`.

## Tags

| Tag           | Description   |
|---------------|---------------|
| `go`          | All tasks     |
| `go:install`  | Toolchain     |
| `go:security` | govulncheck   |

## Example Playbook

```yaml
- hosts: workstations
  roles:
    - role: marcstraube.common.go
      vars:
        go_enabled: true
        go_govulncheck_enabled: true
```

## Testing

```bash
cd roles/go
molecule test
```

The scenario installs the toolchain on all four platforms and govulncheck
where it is packaged.

## References

- [Go](https://go.dev/doc/install)
- [govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck)

## License

MIT

## Author

Marc Straube
