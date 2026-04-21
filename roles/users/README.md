# marcstraube.common.users

Manage system users, groups, SSH authorized keys, and profile images.

## Description

- Group creation with optional GID and system flag
- User creation with full control over UID, shell, home, groups, password
- Root user special handling (password/shell only, no home creation)
- Shell binary validation with fallback to `/bin/bash`
- SSH authorized key deployment with exclusive mode
- SSH key generation (ed25519/rsa)
- Password auto-hashing (plaintext, pre-hashed, or locked)
- Random password generation with optional file storage
- Profile images via AccountsService with optional `~/.face` symlink
- Exclusive mode: remove unlisted users (with keep-list safety)

## Requirements

- `ansible.posix` collection (for `authorized_key` module)

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
| `users_enabled` | `true`  | Enable the users role |

### Groups

| Variable       | Default | Description                                         |
|----------------|---------|-----------------------------------------------------|
| `users_groups` | `[]`    | List of groups to create (name, gid, system, state) |

### Users

| Variable                        | Default     | Description                                            |
|---------------------------------|-------------|--------------------------------------------------------|
| `users_list`                    | `[]`        | List of users to manage (see defaults for full schema) |
| `users_default_shell`           | `/bin/bash` | Default shell for new users                            |
| `users_default_home_base`       | `/home`     | Default home base directory                            |
| `users_default_skeleton`        | `/etc/skel` | Default skeleton directory                             |
| `users_default_create_home`     | `true`      | Create home directory by default                       |
| `users_default_group`           | `''`        | Default primary group (empty = username)               |
| `users_default_groups`          | `[]`        | Default additional groups                              |
| `users_default_append_groups`   | `true`      | Append to groups (vs replace)                          |
| `users_default_update_password` | `always`    | Password update policy: `on_create`, `always`          |
| `users_default_ssh_key_type`    | `ed25519`   | Default SSH key type for generated keys                |

### SSH Key Management

| Variable                         | Default           | Description                       |
|----------------------------------|-------------------|-----------------------------------|
| `users_ssh_authorized_keys_dir`  | `.ssh`            | SSH authorized_keys directory     |
| `users_ssh_authorized_keys_file` | `authorized_keys` | Authorized keys filename          |
| `users_ssh_dir_mode`             | `0700`            | SSH directory permissions         |
| `users_ssh_authorized_keys_mode` | `0600`            | Authorized keys file permissions  |
| `users_create_ssh_dir`           | `true`            | Create `.ssh` directory for users |

### Password Policy

| Variable                         | Default   | Description                                             |
|----------------------------------|-----------|---------------------------------------------------------|
| `users_password_hash_algo`       | `sha512`  | Hash algorithm (`sha512`, `sha256`)                     |
| `users_password_salt`            | vault ref | Fixed salt for idempotent hashing                       |
| `users_generate_random_password` | `false`   | Generate random password for users without one          |
| `users_random_password_length`   | `24`      | Random password length                                  |
| `users_password_store_file`      | `''`      | File to store generated passwords (empty = don't store) |

### Home Directory

| Variable          | Default | Description                |
|-------------------|---------|----------------------------|
| `users_home_mode` | `0700`  | Home directory permissions |

### User Removal

| Variable             | Default | Description                                |
|----------------------|---------|--------------------------------------------|
| `users_remove_home`  | `false` | Remove home directory when user is removed |
| `users_force_remove` | `false` | Force removal even if user is logged in    |

### Profile Images

| Variable                            | Default                          | Description                     |
|-------------------------------------|----------------------------------|---------------------------------|
| `users_profile_images_enabled`      | `false`                          | Enable profile image deployment |
| `users_profile_images_dir`          | `/var/lib/AccountsService/icons` | Profile images directory        |
| `users_profile_images_face_symlink` | `false`                          | Create `~/.face` symlink        |

### Exclusive Mode

| Variable          | Default                        | Description                             |
|-------------------|--------------------------------|-----------------------------------------|
| `users_exclusive` | `false`                        | Remove users not in `users_list`        |
| `users_keep`      | `[root, nobody, ansible_user]` | Users to never remove in exclusive mode |

### User Schema

Each entry in `users_list` supports:

```yaml
- name: 'username'          # Required
  comment: 'Full Name'
  default: true              # Use this user's password for become_password
  uid: 1001
  group: 'username'          # Primary group
  groups: ['wheel', 'docker']
  append_groups: true
  shell: '/bin/bash'
  home: '/home/username'
  home_mode: '0700'
  create_home: true
  move_home: false
  skeleton: '/etc/skel'
  password: "{{ vault_users.username }}"
  password_lock: false
  update_password: 'always'  # on_create, always
  expires: -1                # Never (-1) or epoch timestamp
  system: false
  ssh_keys:
    - 'ssh-ed25519 AAAA... user@host'
  ssh_keys_exclusive: false
  generate_ssh_key: false
  ssh_key_type: 'ed25519'
  ssh_key_bits: 4096
  ssh_key_comment: ''
  ssh_key_passphrase: ''
  profile_image: ''          # Path to image file
  state: 'present'           # present, absent
```

## Tags

| Tag                    | Scope                    |
|------------------------|--------------------------|
| `users`                | All role tasks           |
| `users:groups`         | Group management         |
| `users:users`          | User creation/removal    |
| `users:ssh-keys`       | SSH key deployment       |
| `users:profile-images` | Profile image management |

## Example Playbook

```yaml
- name: Include users role
  ansible.builtin.include_role:
    name: marcstraube.common.users
  tags: [users]
  when: users_enabled | default(true) | bool
```

## Testing

```bash
cd roles/users
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

- Root user receives special handling: only password and shell are updated
- Shell validation checks if the binary exists, falls back to `/bin/bash`
- Password hashing is automatic: plaintext passwords are hashed with the configured algorithm
- Use `users_password_salt` (stored in vault) for idempotent password hashing
- Profile images use AccountsService and optionally `~/.face` symlinks for DM compatibility
- Exclusive mode only affects regular users (UID 1000-59999), never system accounts

## References

- [useradd(8)](https://man7.org/linux/man-pages/man8/useradd.8.html) — user account creation man page
- [login.defs(5)](https://man7.org/linux/man-pages/man5/login.defs.5.html) — shadow password suite configuration

## License

MIT

## Author

Marc Straube
