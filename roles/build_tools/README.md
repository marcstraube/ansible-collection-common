# marcstraube.common.build_tools

Install build systems (make, CMake, Meson, Ninja) and the C/C++ toolchain with
its helper tools.

## Requirements

- ansible-core >= 2.19
- Collections: `community.general`, `kewlfft.aur` (codelldb on Arch Linux)
- On Arch Linux: an `aur_builder` user for the AUR package (set up by
  `package_management`)
- On EL: CRB for meson and ninja (`dnf_powertools_enabled` in
  `package_management`) and EPEL for ccache, cppcheck and bear (enabled by
  `package_management`)

## Supported Platforms

| Platform                   | Notes                                   |
|----------------------------|-----------------------------------------|
| Arch Linux                 |                                         |
| Debian Trixie              |                                         |
| EL 9 (Rocky, Alma, RHEL)   | valgrind comes with Development Tools   |
| EL 10 (Rocky, Alma, RHEL)  | valgrind as EL 9; bear is not packaged  |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable              | Default | Description                 |
|-----------------------|---------|-----------------------------|
| `build_tools_enabled` | `false` | Enable the build_tools role |

### Build Systems

| Variable                    | Default | Description                                                |
|-----------------------------|---------|------------------------------------------------------------|
| `build_tools_make_enabled`  | `true`  | Core build tools (development group of the platform, make) |
| `build_tools_cmake_enabled` | `true`  | CMake                                                      |
| `build_tools_meson_enabled` | `true`  | Meson                                                      |
| `build_tools_ninja_enabled` | `true`  | Ninja                                                      |

The core build tools are `base-devel` on Arch Linux, `build-essential` on
Debian and the `Development Tools` group on EL, each with `make`. On EL, meson
and ninja come from the CRB repository.

### C/C++ Toolchain

| Variable                         | Default | Description                                               |
|----------------------------------|---------|-----------------------------------------------------------|
| `build_tools_c_enabled`          | `false` | clang with clangd, clang-format, clang-tidy; gdb; pkgconf |
| `build_tools_c_codelldb_enabled` | `false` | codelldb LLDB debug adapter (Arch Linux AUR)              |
| `build_tools_c_ccache_enabled`   | `false` | ccache compiler cache                                     |
| `build_tools_c_valgrind_enabled` | `false` | valgrind memory debugger                                  |
| `build_tools_c_cppcheck_enabled` | `false` | cppcheck static analysis                                  |
| `build_tools_c_bear_enabled`     | `false` | bear, compile_commands.json for non-CMake builds          |

Tools a platform does not package are skipped: codelldb exists only as an AUR
package (upstream ships a VSIX archive), bear is missing from EPEL 10, and
valgrind on EL arrives as a member of the Development Tools group, so the
toggle is a no-op there. The role installs and never removes; disabling a
toggle leaves an installed tool in place.

## Tags

| Tag                   | Description                      |
|-----------------------|----------------------------------|
| `build-tools`         | All tasks                        |
| `build-tools:install` | Core build tools, build systems  |
| `build-tools:c`       | C/C++ toolchain and helper tools |

## Example Playbook

```yaml
- hosts: workstations
  roles:
    - role: marcstraube.common.build_tools
      vars:
        build_tools_enabled: true
        build_tools_c_enabled: true
        build_tools_c_ccache_enabled: true
        build_tools_c_valgrind_enabled: true
```

## Testing

```bash
cd roles/build_tools
molecule test
```

The scenario enables every toggle and checks that the binaries run on all four
platforms, skipping the tools a platform does not package.

## References

- [CMake](https://cmake.org/), [Meson](https://mesonbuild.com/),
  [Ninja](https://ninja-build.org/)
- [Clang](https://clang.llvm.org/), [ccache](https://ccache.dev/),
  [Valgrind](https://valgrind.org/), [Cppcheck](https://cppcheck.sourceforge.io/),
  [Bear](https://github.com/rizsotto/Bear)
- [codelldb](https://github.com/vadimcn/codelldb)

## License

MIT

## Author

Marc Straube
