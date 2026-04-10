# marcstraube.common.nodejs

Install and configure the Node.js runtime. Supports installation via system
packages, NodeSource repository, or nvm (Node Version Manager).

## Description

This role manages Node.js installation and configuration across multiple
platforms, supporting system packages, NodeSource repositories, and nvm
(Node Version Manager). It handles LTS version mapping on Arch Linux and
smart AppStream/NodeSource selection on RedHat-family systems.

## Requirements

- ansible-core >= 2.17

## Supported Platforms

- Arch Linux
- Debian Trixie
- Rocky Linux 9 / 10 (RHEL compatible)

## Installation Methods

| Method       | Arch                           | Debian               | Rocky                               |
| ------------ | ------------------------------ | -------------------- | ----------------------------------- |
| `package`    | pacman (LTS-aware)             | apt (distro version) | dnf (distro version)                |
| `nodesource` | auto-fallback to `package`     | NodeSource repo      | AppStream stream or NodeSource repo |
| `nvm`        | system package + per-user init | curl install script  | curl install script                 |

### Arch Linux Version Mapping

When using `package` or `nodesource` (which falls back to `package` on Arch),
the role maps `nodejs_version` to the correct Arch package:

| `nodejs_version`      | Arch Package         | Release           |
| --------------------- | -------------------- | ----------------- |
| `20`                  | `nodejs-lts-iron`    | LTS (Maintenance) |
| `22`                  | `nodejs-lts-jod`     | LTS (Maintenance) |
| `24`                  | `nodejs-lts-krypton` | LTS (Active)      |
| any other (e.g. `25`) | `nodejs`             | Current (non-LTS) |

### RedHat/Rocky Smart Version Handling

On RedHat-family systems with `nodesource` method, the role:

1. Checks if the requested version is already installed
2. If not, checks if AppStream provides the requested module stream
3. If AppStream has it: enables the stream and installs from AppStream
4. If not: disables the module, adds the NodeSource repository, and installs

## Role Variables

### Installation

| Variable                | Default        | Description                                                              |
| ----------------------- | -------------- | ------------------------------------------------------------------------ |
| `nodejs_enabled`        | `true`         | Enable/disable the role                                                  |
| `nodejs_install_method` | `'nodesource'` | Installation method: `nodesource`, `package`, or `nvm`                   |
| `nodejs_version`        | `'24'`         | Node.js major version — LTS (`'20'`, `'22'`, `'24'`) or Current (`'25'`) |

### npm Configuration

| Variable            | Default | Description                           |
| ------------------- | ------- | ------------------------------------- |
| `nodejs_npm_proxy`  | `''`    | HTTP(S) proxy for npm                 |
| `nodejs_npm_config` | `{}`    | Additional npm config key-value pairs |

### Package Managers

| Variable                  | Default | Description                                          |
| ------------------------- | ------- | ---------------------------------------------------- |
| `nodejs_package_managers` | `[]`    | Additional package managers via npm (`pnpm`, `yarn`) |

### nvm Settings

| Variable                  | Default    | Description                                                 |
| ------------------------- | ---------- | ----------------------------------------------------------- |
| `nodejs_nvm_version`      | `'latest'` | nvm version to install (`'latest'` or tag like `'v0.40.1'`) |
| `nodejs_nvm_users`        | `[]`       | Users to install nvm for (required when method is `nvm`)    |
| `nodejs_nvm_node_version` | `'lts'`    | Node.js version via nvm (`'lts'`, `'current'`, or specific) |

## Tags

| Tag                | Scope                                  |
| ------------------ | -------------------------------------- |
| `nodejs`           | All role tasks                         |
| `nodejs:install`   | Installation and verification          |
| `nodejs:configure` | npm configuration and package managers |

## Example Playbook

```yaml
- name: Install Node.js
  ansible.builtin.include_role:
    name: marcstraube.common.nodejs
  vars:
    nodejs_version: '24'
    nodejs_package_managers:
      - yarn
```

### Install Current (non-LTS) release

```yaml
- name: Install Node.js Current
  ansible.builtin.include_role:
    name: marcstraube.common.nodejs
  vars:
    nodejs_version: '25'
```

On Arch this installs the `nodejs` package (rolling Current). On Debian/Rocky
it pulls the matching version from NodeSource.

## Testing

```bash
cd roles/nodejs
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## License

MIT

## Author

Marc Straube
