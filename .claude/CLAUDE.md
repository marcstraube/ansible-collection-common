# Claude Code — marcstraube.common Collection

## Overview

Shared Ansible roles for all system types (desktops, laptops, servers).
Multi-OS support: Arch Linux (primary), Debian Trixie, Rocky Linux 9/10.

## Coding Standards (ansible-core 2.17+)

- **Facts:** `ansible_facts['os_family']` — never `ansible_os_family` (deprecated)
- **Modules:** Always FQCN (`ansible.builtin.*`, `community.general.*`, `kewlfft.aur.*`)
- **Roles:** `ansible.builtin.include_role` — never `import_role`
- **Tasks:** `ansible.builtin.include_tasks` — never `import_tasks` for conditional dispatch
- **Tag propagation:** Always `apply: tags:` when using `include_tasks` inside blocks
- **Variables:** `<role>_<setting>`, internal: `__<role>_<setting>` (double underscore)
- **Booleans:** `<role>_enabled | default(true/false) | bool`
- **Templates:** `{{ ansible_managed | comment }}` header
- **Comments:** English only

### YAML Quoting Rules for Facts

| Context | Syntax | Example |
| --------- | -------- | --------- |
| `when:` clause (bare Jinja2) | Single quotes inside | `ansible_facts['os_family']` |
| Single-quoted YAML string | Double quotes inside | `'{{ ansible_facts["os_family"] }}'` |
| Double-quoted YAML string | Single quotes inside | `"{{ ansible_facts['os_family'] }}"` |
| Block scalar (`>`, `>-`, `\|`) | Single quotes inside | `ansible_facts['os_family']` |

## Role Conventions

### Directory Structure

```text
roles/<role_name>/
├── defaults/main.yml       # All variables with sensible defaults
├── handlers/main.yml       # Service restart/reload handlers
├── meta/main.yml           # Galaxy metadata, dependencies
├── tasks/
│   ├── main.yml            # Dispatcher: load OS vars, include OS tasks
│   ├── install.yml         # Installation tasks
│   ├── configure.yml       # Configuration tasks
│   ├── service.yml         # Service management
│   ├── btrfs.yml           # BTRFS optimizations (if applicable)
│   ├── firewall.yml        # Firewalld integration (if applicable)
│   └── apparmor.yml        # AppArmor integration (if applicable)
├── templates/*.j2          # Jinja2 templates
├── vars/
│   ├── Archlinux.yml       # OS-specific package names, paths, services
│   ├── Debian.yml
│   └── RedHat.yml          # Always latest (Rocky 10), overrides for older
└── molecule/default/       # Tests
    ├── molecule.yml
    ├── converge.yml
    └── verify.yml
```

### Variable Naming

- Role variables: `<role_name>_<setting>` (e.g., `chrony_ntp_servers`)
- Internal/OS-specific variables (vars/): `__<role_name>_<setting>` (double underscore prefix)
- Boolean toggles: `<role_name>_enabled`, `<role_name>_<feature>_enabled`
- Per-user config: `<role>_users` list with `mode: managed|initial|disabled`

### Task Patterns

- `main.yml` loads OS-specific vars with `include_vars` + `with_first_found`
- Use `include_tasks` (not `import_tasks`) for conditional OS dispatch
- `import_tasks` only for unconditional static includes in main.yml
- Always add `apply: tags:` when using `include_tasks` inside a block
- Use `ansible.builtin.` FQCN for all modules

### Templates

- Always include `{{ ansible_managed | comment }}` at the top
- Blank lines and descriptive comments go INSIDE `{% if %}` blocks, never before them
- No static blank line between `{% endif %}` and next `{% if %}`

### Examples in defaults/main.yml

- Always use generic placeholder data (`johndoe`, `john@example.com`)
- Never use real usernames or email addresses

## Testing

- Molecule with Podman containers, systemd, privileged mode
- Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10
- Tests verify: packages, configs, services, permissions, content
- Run one role at a time (RAM constraints with 4 containers)

## Security

- Vault files (`vault.yml`) are never accessed — only `vault.yml.example` templates
- Use `{{ vault_* }}` variable references, never hardcode secrets
