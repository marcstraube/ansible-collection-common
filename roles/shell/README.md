# shell

Install and configure shells (Zsh, Fish, Bash) and Starship prompt with modern CLI tools.

## Requirements

None.

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

| Variable        | Default | Description           |
|-----------------|---------|-----------------------|
| `shell_enabled` | `true`  | Enable the shell role |

### Default Shell

| Variable                 | Default     | Description                                               |
|--------------------------|-------------|-----------------------------------------------------------|
| `shell_default`          | `'bash'`    | Default shell for users: zsh, fish, bash                  |
| `shell_users`            | `[]`        | Users to change shell (empty = all from users_list)       |
| `shell_user_config_mode` | `'initial'` | User config mode: managed, initial (if absent), disabled  |

### Zsh

| Variable                 | Default    | Description                                          |
|--------------------------|------------|------------------------------------------------------|
| `shell_zsh_enabled`      | `false`    | Enable Zsh installation                              |
| `shell_zsh_framework`    | `'native'` | Zsh framework: native or ohmyzsh                     |
| `shell_zsh_plugins`      | `[...]`    | Zsh plugins (native framework only)                  |
| `shell_zsh_grml_enabled` | `false`    | Enable grml-zsh-config (native framework only)       |

### Oh My Zsh

Only used when `shell_zsh_framework: 'ohmyzsh'`. Installs Oh My Zsh per-user via git
clone and deploys a managed `.zshrc`. Requires `git` to be installed.

| Variable                        | Default          | Description                                    |
|---------------------------------|------------------|------------------------------------------------|
| `shell_ohmyzsh_theme`           | `'robbyrussell'` | Oh My Zsh theme (set to `''` for Starship)     |
| `shell_ohmyzsh_plugins`         | `['git']`        | Built-in Oh My Zsh plugins to activate         |
| `shell_ohmyzsh_custom_plugins`  | `[]`             | Third-party plugins (list of name/repo dicts)  |

Custom plugins example:

```yaml
shell_ohmyzsh_custom_plugins:
  - name: 'zsh-autosuggestions'
    repo: 'https://github.com/zsh-users/zsh-autosuggestions.git'
  - name: 'zsh-syntax-highlighting'
    repo: 'https://github.com/zsh-users/zsh-syntax-highlighting.git'
```

When `shell_starship_enabled` is `true`, the deployed `.zshrc` sets `ZSH_THEME=""`
and adds Starship init automatically.

### Fish

| Variable             | Default | Description              |
|----------------------|---------|--------------------------|
| `shell_fish_enabled` | `false` | Enable Fish installation |

### Bash

| Variable                        | Default | Description            |
|---------------------------------|---------|------------------------|
| `shell_bash_completion_enabled` | `true`  | Enable bash-completion |

### Starship

| Variable                        | Default           | Description                                |
|---------------------------------|-------------------|--------------------------------------------|
| `shell_starship_enabled`        | `true`            | Enable Starship prompt installation        |
| `shell_starship_config_enabled` | `false`           | Enable Starship config deployment          |
| `shell_starship_preset`         | `''`              | Starship preset (empty = minimal default)  |
| `shell_starship_shells`         | `['zsh', 'bash']` | Configure Starship for these shells        |

When `shell_starship_config_enabled` is `true`, the role deploys
`~/.config/starship.toml` per user (from `shell_users` or `users_list`).
With a preset set, content is rendered via `starship preset <name>`. With
an empty preset, a minimal placeholder is deployed. Deployment honours
`shell_user_config_mode`: `'managed'` reconciles every run, `'initial'`
only deploys if the file is absent, `'disabled'` skips entirely.

Valid preset names (source: `starship preset --list`):
`bracketed-segments`, `catppuccin-powerline`, `gruvbox-rainbow`,
`jetpack`, `nerd-font-symbols`, `no-empty-icons`, `no-nerd-font`,
`no-runtime-versions`, `pastel-powerline`, `plain-text-symbols`,
`pure-preset`, `tokyo-night`. An invalid preset value fails the
role at start with a clear assertion message.

`shell_starship_shells` controls which shell rcfiles get a
starship init line appended via `blockinfile` (with marker
`# ANSIBLE MANAGED — starship init`). Independent from
`shell_starship_config_enabled` — a user can opt into starship
init using starship defaults without deploying a custom config.

| Shell  | Target file                  | Init line                          |
|--------|------------------------------|------------------------------------|
| `bash` | `~/.bashrc`                  | `eval "$(starship init bash)"`     |
| `zsh`  | `~/.zshrc`                   | `eval "$(starship init zsh)"`      |
| `fish` | `~/.config/fish/config.fish` | `starship init fish \| source`     |

The zsh init line is skipped when `shell_zsh_framework: 'ohmyzsh'` —
the Oh My Zsh `.zshrc` template already includes starship init when
both are enabled, so adding it again would duplicate.

Init lines honour `shell_user_config_mode`: `'managed'` reconciles
the marker block on every run; `'initial'` deploys the block on
first run and leaves the file alone if the marker is already present
(preserving any user modifications inside it); `'disabled'` skips.

### Additional Tools

| Variable                 | Default           | Description                                  |
|--------------------------|-------------------|----------------------------------------------|
| `shell_fzf_enabled`      | `true`            | Enable fzf (fuzzy finder)                    |
| `shell_zoxide_enabled`   | `false`           | Enable zoxide (smarter cd)                   |
| `shell_zoxide_shells`    | `['zsh', 'bash']` | Configure zoxide init for these shells       |
| `shell_eza_enabled`      | `false`           | Enable eza (modern ls)                       |
| `shell_bat_enabled`      | `false`           | Enable bat (cat with syntax highlighting)    |
| `shell_fd_enabled`       | `false`           | Enable fd (modern find)                      |
| `shell_ripgrep_enabled`  | `false`           | Enable ripgrep (modern grep)                 |
| `shell_tldr_enabled`     | `false`           | Enable tldr (simplified man pages)           |

`shell_zoxide_shells` controls which shell rcfiles get a zoxide init
line appended via `blockinfile` (with marker
`# ANSIBLE MANAGED — zoxide init`). Without an init line, the
`zoxide` package is installed but unusable — `eval "$(zoxide init <shell>)"`
must be sourced for the `z` command to work.

| Shell  | Target file                  | Init line                          |
|--------|------------------------------|------------------------------------|
| `bash` | `~/.bashrc`                  | `eval "$(zoxide init bash)"`       |
| `zsh`  | `~/.zshrc`                   | `eval "$(zoxide init zsh)"`        |
| `fish` | `~/.config/fish/config.fish` | `zoxide init fish \| source`       |

The zsh init line is skipped when `shell_zsh_framework: 'ohmyzsh'` —
the Oh My Zsh `.zshrc` template already includes zoxide init when
`shell_zoxide_enabled` is true, so adding it again would duplicate.

Init lines honour `shell_user_config_mode`: `'managed'` reconciles
the marker block on every run; `'initial'` deploys the block on
first run and leaves the file alone if the marker is already present
(preserving any user modifications inside it); `'disabled'` skips.

On platforms where `zoxide` is not packaged (currently EL 10 — not in
EPEL 10), `shell_zoxide_enabled` is a silent no-op: neither the
package nor the init line is deployed.

## Tags

| Tag               | Description                    |
|-------------------|--------------------------------|
| `shell`           | All shell tasks                |
| `shell:install`   | Package installation           |
| `shell:ohmyzsh`   | Oh My Zsh setup and config     |
| `shell:users`     | User shell configuration       |
| `shell:starship`  | Starship per-user config       |
| `shell:zoxide`    | Zoxide per-user init           |

## Example Playbook

```yaml
- hosts: workstations
  become: true
  tasks:
    - name: Include shell role
      ansible.builtin.include_role:
        name: marcstraube.common.shell
      tags:
        - shell
      when: shell_enabled | default(true) | bool
```

## Testing

```bash
cd collections/ansible_collections/marcstraube/common
molecule test -s default -- --limit shell-archlinux
```

## References

- [Bash](https://www.gnu.org/software/bash/) — GNU Bourne Again SHell
- [Zsh](https://www.zsh.org/) — Z Shell — extended Bourne shell with many improvements
- [Fish](https://fishshell.com/) — Smart and user-friendly command line shell
- [Oh My Zsh](https://ohmyz.sh/) — Community-driven framework for managing Zsh configuration
- [Starship](https://starship.rs/) — Minimal, fast, customizable cross-shell prompt
- [Starship Presets](https://starship.rs/presets/) — Curated configuration presets (`starship preset --list`)
- [grml-zsh-config](https://grml.org/zsh/) — Feature-rich Zsh setup from the GRML project
- [fzf](https://github.com/junegunn/fzf) — Command-line fuzzy finder
- [zoxide](https://github.com/ajeetdsouza/zoxide) — Smarter `cd` command
- [eza](https://github.com/eza-community/eza) — Modern replacement for `ls`
- [bat](https://github.com/sharkdp/bat) — `cat` clone with syntax highlighting and Git integration
- [fd](https://github.com/sharkdp/fd) — Simple, fast, user-friendly alternative to `find`
- [ripgrep](https://github.com/BurntSushi/ripgrep) — Recursive `grep` alternative respecting gitignore
- [tldr](https://tldr.sh/) — Simplified, community-driven man pages

## License

MIT

## Author

Marc Straube
