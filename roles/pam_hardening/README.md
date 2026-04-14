# marcstraube.common.pam_hardening

Harden PAM (Pluggable Authentication Modules) configuration across Arch Linux,
Debian Trixie, Rocky 9, and Rocky 10.

## Description

Comprehensive PAM hardening role covering password quality (pwquality), account
lockout (faillock), password history, session limits, access control, umask,
password aging, password hashing (yescrypt/SHA-512), null password control, login
delay, secure TTY, and optional TOTP 2FA via Google Authenticator.

Each OS family uses its native PAM stack management tool: pambase on Arch,
pam-auth-update on Debian, and authselect on Rocky/RHEL.

## Requirements

- ansible-core >= 2.17
- No additional collections required

## Supported Platforms

| Platform                  | Notes |
|---------------------------|-------|
| Arch Linux                |       |
| Debian Trixie             |       |
| EL 9 (Rocky, Alma, RHEL)  |       |
| EL 10 (Rocky, Alma, RHEL) |       |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

See [`defaults/main.yml`](defaults/main.yml) for all variables with descriptions.

Key toggles:

| Variable                     | Default | Description                          |
|------------------------------|---------|--------------------------------------|
| `pam_hardening_enabled`      | `true`  | Master toggle                        |
| `pam_pwquality_enabled`      | `true`  | Password quality checking            |
| `pam_faillock_enabled`       | `true`  | Account lockout                      |
| `pam_limits_enabled`         | `true`  | Session limits                       |
| `pam_access_enabled`         | `true`  | Access control                       |
| `pam_password_aging_enabled` | `false` | Password aging (NIST recommends off) |
| `pam_securetty_enabled`      | `false` | Secure TTY (deprecated)              |
| `pam_2fa_enabled`            | `false` | TOTP two-factor auth                 |
| `pam_permit_empty_passwords` | `false` | Null password auth                   |

## Tags

| Tag                       | Scope                        |
|---------------------------|------------------------------|
| `pam-hardening`           | All role tasks               |
| `pam-hardening:install`   | Package installation         |
| `pam-hardening:configure` | PAM configuration deployment |

## Example Playbook

```yaml
- name: Include pam_hardening role
  ansible.builtin.include_role:
    name: marcstraube.common.pam_hardening
  tags: [security, pam-hardening]
  when: pam_hardening_enabled | default(true) | bool
```

## Testing

```bash
cd roles/pam_hardening
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

### Platform-Specific PAM Stack Integration

| Component | Arch Linux             | Debian Trixie               | Rocky 9/10                |
|-----------|------------------------|-----------------------------|---------------------------|
| faillock  | In pambase (no change) | lineinfile common-auth      | authselect with-faillock  |
| pwquality | lineinfile system-auth | pam-auth-update (auto)      | In authselect profile     |
| pwhistory | lineinfile system-auth | pam_unix remember=          | authselect with-pwhistory |
| limits    | In pambase (no change) | lineinfile common-session   | In authselect profile     |
| access    | In pambase (no change) | lineinfile common-account   | authselect with-pamaccess |
| umask     | In pambase (no change) | In common-session (default) | In authselect profile     |
| nullok    | replace system-auth    | replace common-auth         | authselect without-nullok |

### Arch Linux

Arch pambase (20230918+) already includes pam_faillock, pam_limits, pam_access,
and pam_umask in the default PAM stack. This role only adds pam_pwquality and
pam_pwhistory to the password stack and deploys configuration files.

### Debian Trixie

Debian uses `pam-auth-update` to manage `common-*` PAM files. This role uses
lineinfile for faillock/access/limits integration. Note that `pam-auth-update
--package` (triggered by PAM package upgrades) may regenerate common-auth,
requiring a re-run of this role.

### Rocky 9/10

Rocky uses `authselect` for PAM stack management. This role enables features
via `authselect enable-feature` and deploys configuration files in
`/etc/security/` which are not managed by authselect.

## License

MIT

## Author

Marc Straube
