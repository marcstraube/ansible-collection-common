# Contributing to marcstraube.common

Thank you for your interest in contributing! This document provides guidelines
and instructions for contributing to this collection.

## Development Setup

### Prerequisites

- Python 3.13+
- ansible-core >= 2.17
- Podman (for Molecule tests)
- Git
- pre-commit

### Clone and Setup

```bash
git clone https://github.com/marcstraube/ansible-collection-common.git
cd ansible-collection-common

pip install ansible-core ansible-lint molecule molecule-plugins[podman] yamllint pre-commit

ansible-galaxy collection install -r galaxy.yml

pre-commit install
```

## Pre-commit Hooks

```bash
# Run all hooks
pre-commit run --all-files

# Run specific hook
pre-commit run ansible-lint --all-files
pre-commit run yamllint --all-files
```

### Configured Hooks

- **ansible-lint** - Ansible best practices and syntax
- **yamllint** - YAML syntax and formatting
- **markdownlint** - Markdown formatting
- **trailing-whitespace** - Remove trailing whitespace
- **end-of-file-fixer** - Ensure files end with newline
- **check-yaml** - Validate YAML syntax
- **detect-secrets** - Prevent accidental secret commits

## Running Tests

### Linting

```bash
ansible-lint
yamllint -c .yamllint .
```

### Molecule Tests

```bash
# Test a specific role
cd roles/chrony
molecule test

# Step by step (for debugging)
molecule create
molecule converge
molecule verify
molecule destroy

# Login to container
molecule login
```

## Code Style

### Ansible Conventions

- FQCN for all modules: `ansible.builtin.*`, `community.general.*`
- Facts syntax: `ansible_facts['key']` (never `ansible_key`)
- `include_tasks` for conditional dispatch, `import_tasks` for static includes
- Tag propagation: `apply: tags:` on `include_tasks`
- Variables: `<role>_<setting>`, internal: `__<role>_<setting>`
- Boolean toggles: `<role>_enabled | default(true) | bool`
- Templates: `{{ ansible_managed | comment }}` header
- Comments: English only

### Role Structure

```text
roles/<role>/
├── defaults/main.yml       # All variables with defaults
├── handlers/main.yml       # Service handlers
├── meta/main.yml           # Galaxy metadata
├── tasks/
│   ├── main.yml            # Dispatcher: load vars, include tasks
│   ├── install.yml         # Installation
│   ├── configure.yml       # Configuration
│   └── archlinux/*.yml     # OS-specific tasks (if needed)
├── templates/*.j2          # Jinja2 templates
├── vars/
│   ├── Archlinux.yml       # OS-specific package names, paths
│   ├── Debian.yml
│   └── RedHat.yml
└── molecule/default/       # Tests
    ├── molecule.yml
    ├── converge.yml
    └── verify.yml
```

### Variable Naming

```yaml
# Role variables (defaults/main.yml)
chrony_ntp_servers: []
chrony_enabled: true

# Internal/OS-specific (vars/*.yml)
__chrony_package: 'chrony'
__chrony_service: 'chronyd'
```

### Examples in defaults/main.yml

Always use generic placeholder data in commented examples:

```yaml
# Good
#   - username: 'johndoe'
#     email: 'john@example.com'

# Bad — never use real usernames/emails
#   - username: 'mst'
```

## Pull Request Process

1. Run pre-commit hooks: `pre-commit run --all-files`
2. Test changes: `molecule test` on at least one platform
3. Update documentation if needed
4. Use conventional commit format:
   - `feat(chrony): add NTS support`
   - `fix(openssh): correct TLS configuration`
   - `refactor(users): simplify group management`

## License

By contributing, you agree that your contributions will be licensed under
the MIT License.
