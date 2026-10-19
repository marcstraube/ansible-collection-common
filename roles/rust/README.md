# marcstraube.common.rust

Install the Rust toolchain from distribution packages or rustup, with
toolchain components and cargo-audit.

## Requirements

- ansible-core >= 2.19
- Collections: `community.general`
- `rust_install_method: 'rustup'` needs the distribution's rustup package
  (Arch Linux, Debian); EL has none and falls back to `package`

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

| Variable       | Default | Description          |
|----------------|---------|----------------------|
| `rust_enabled` | `false` | Enable the rust role |

### Installation

| Variable              | Default                 | Description                           |
|-----------------------|-------------------------|---------------------------------------|
| `rust_install_method` | `package`               | Installation method (package, rustup) |
| `rust_components`     | `['rustfmt', 'clippy']` | Toolchain components to install       |

`package` installs the distribution's toolchain system-wide: `rust` on Arch
Linux, `rustc` and `cargo` on Debian and EL. Components map to distribution
packages; Arch Linux bundles rustfmt and clippy in `rust` and ships
rust-analyzer as its own package. Valid components: `rustfmt`, `clippy`,
`rust-analyzer`.

`rustup` installs the distribution's rustup package and a per-user toolchain
for every user in `rust_rustup_users`, with the components added through
`rustup component add`. On Arch Linux the rustup package replaces `rust` and
`cargo`; switching an installed host from `package` to `rustup` means
removing `rust` first. EL ships no rustup package, so the role falls back to
`package` there.

### rustup Settings

Only used when `rust_install_method` is `rustup`.

| Variable                | Default  | Description                                       |
|-------------------------|----------|---------------------------------------------------|
| `rust_rustup_users`     | `[]`     | Users who get a rustup toolchain                  |
| `rust_rustup_toolchain` | `stable` | Toolchain to install and set as default           |

### Security Tooling

| Variable                   | Default | Description                                        |
|----------------------------|---------|----------------------------------------------------|
| `rust_cargo_audit_enabled` | `false` | Install cargo-audit (RustSec advisory audit)       |

cargo-audit is packaged on Arch Linux only. Debian and EL get a notice and
can install it with `cargo install cargo-audit`.

## Tags

| Tag             | Description                      |
|-----------------|----------------------------------|
| `rust`          | All tasks                        |
| `rust:install`  | Toolchain and components         |
| `rust:security` | cargo-audit                      |

## Example Playbook

```yaml
- hosts: workstations
  roles:
    - role: marcstraube.common.rust
      vars:
        rust_enabled: true
        rust_components:
          - 'rustfmt'
          - 'clippy'
          - 'rust-analyzer'
        rust_cargo_audit_enabled: true
```

Per-user toolchains through rustup:

```yaml
- hosts: workstations
  roles:
    - role: marcstraube.common.rust
      vars:
        rust_enabled: true
        rust_install_method: 'rustup'
        rust_rustup_users:
          - 'johndoe'
        rust_rustup_toolchain: 'stable'
```

## Testing

```bash
cd roles/rust
molecule test
```

The scenario installs the toolchain and all three components from
distribution packages on all four platforms and cargo-audit on Arch Linux.
The rustup method is not part of the scenario.

## References

- [Rust toolchain](https://www.rust-lang.org/tools/install)
- [rustup](https://rust-lang.github.io/rustup/)
- [cargo-audit](https://github.com/rustsec/rustsec/tree/main/cargo-audit)

## License

MIT

## Author

Marc Straube
