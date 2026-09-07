# git

Installs and configures **git** together with the surrounding version control
tooling — `git-lfs`, the GitHub CLI (`gh`) and the GitLab CLI (`glab`) — and
applies per-user git identity and signing configuration.

## Description

git is baseline tooling on servers and workstations alike, which is why this
role lives in `common` next to the other language and CLI tooling roles rather
than in a workstation-specific collection.

Each tool has its own toggle. Where a distribution ships no upstream package
for a tool, the internal package variable is an empty string and the task
becomes a no-op instead of failing the run.

Per-user configuration covers `user.name`, `user.email`, `user.signingkey` and
`commit.gpgsign`, plus any settings given in `git_config`. Users are taken from
`git_users`; when that list is empty, it is derived from `users_list` entries
carrying a `git` attribute, so the git identity lives on the account it belongs
to.

## Requirements

- `community.general` collection (for the `git_config` module)
- EPEL enabled on EL hosts for `gh` and `git-lfs`

## Supported Platforms

| Platform                  | Notes                                             |
|---------------------------|---------------------------------------------------|
| Arch Linux                | Includes `git-delta`; `glab` available            |
| Debian Trixie             | `glab` not packaged                               |
| EL 9 (Rocky, Alma, RHEL)  | `gh` and `git-lfs` need EPEL; `glab` not packaged |
| EL 10 (Rocky, Alma, RHEL) | `gh` and `git-lfs` need EPEL; `glab` not packaged |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable      | Default | Description         |
|---------------|---------|---------------------|
| `git_enabled` | `true`  | Enable the git role |

### Version Control Tooling

| Variable                 | Default | Description                     |
|--------------------------|---------|---------------------------------|
| `git_lfs_enabled`        | `true`  | Install git-lfs                 |
| `git_github_cli_enabled` | `true`  | Install the GitHub CLI (`gh`)   |
| `git_gitlab_cli_enabled` | `true`  | Install the GitLab CLI (`glab`) |

Setting a toggle to `false` removes the package again. Tools without an
upstream package on the target platform are skipped either way.

### Global Configuration

| Variable     | Default   | Description                                      |
|--------------|-----------|--------------------------------------------------|
| `git_config` | See below | Settings applied to each user's global git scope |

```yaml
git_config:
  init.defaultBranch: 'main'
  pull.rebase: 'true'
  push.autoSetupRemote: 'true'
```

Add `core.editor` here to enforce a specific editor; git falls back to the
`EDITOR` environment variable when it is unset.

### Per-User Configuration

| Variable               | Default     | Description                                |
|------------------------|-------------|--------------------------------------------|
| `git_users`            | `[]`        | Users receiving git configuration          |
| `git_user_config_mode` | `'initial'` | Default mode when user entry has no 'mode' |

Each entry accepts `username`, an optional `mode`, and the optional identity
keys `git_name`, `git_email` and `git_signing_key`. Providing `git_signing_key`
also enables `commit.gpgsign` for that user.

| Mode       | Behavior                                         |
|------------|--------------------------------------------------|
| `managed`  | Apply the configuration on every run             |
| `initial`  | Apply only for users created during the same run |
| `disabled` | Skip the user entirely                           |

## Tags

| Tag           | Scope                      |
|---------------|----------------------------|
| `git`         | All tasks                  |
| `git:install` | Package installation       |
| `git:users`   | Per-user config deployment |

## Example Playbook

```yaml
- name: Configure version control tooling
  hosts: all
  become: true

  vars:
    git_gitlab_cli_enabled: false
    git_config:
      init.defaultBranch: 'main'
      pull.rebase: 'true'
      core.editor: 'vim'
    git_users:
      - username: 'johndoe'
        mode: 'managed'
        git_name: 'John Doe'
        git_email: 'johndoe@example.com'
        git_signing_key: 'YOUR_GPG_KEY_ID'

  roles:
    - marcstraube.common.git
```

## Testing

Molecule tests package installation, toggle behavior and per-user
configuration across Arch Linux, Debian Trixie and Rocky Linux 9/10. The
scenario drives the user list through derivation from `users_list` and asserts
that system accounts and entries without a `git` attribute are left untouched.

## Notes

- On Arch Linux the git package set also pulls in `git-delta`, matching the
  package list this role was extracted from.
- `glab` is packaged for Arch only. On Debian and EL the toggle has no effect.

## References

- [git documentation](https://git-scm.com/doc)
- [git-lfs](https://git-lfs.com/)
- [GitHub CLI](https://cli.github.com/manual/)
- [GitLab CLI](https://gitlab.com/gitlab-org/cli)

## License

MIT

## Author

Marc Straube
