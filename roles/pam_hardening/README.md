# marcstraube.common.pam_hardening

Harden PAM (Pluggable Authentication Modules) configuration across Arch Linux,
Debian Trixie, Rocky 9, and Rocky 10.

## Description

- **Password quality** — pwquality.conf with full directive coverage
- **Account lockout** — faillock.conf with automatic PAM stack integration
- **Password history** — pam_pwhistory (Arch/RedHat) or pam_unix remember (Debian)
- **Session limits** — limits.d drop-in with custom rule support
- **Access control** — access.conf with deny-all default
- **Umask** — via login.defs (all platforms)
- **Password aging** — login.defs PASS_MAX_DAYS/MIN_DAYS/WARN_AGE
- **Password hashing** — yescrypt (default) or SHA-512
- **Null password control** — disable nullok across platforms
- **Login delay** — pam_faildelay (Arch/Debian) or FAIL_DELAY (RedHat)
- **Secure TTY** — optional /etc/securetty (deprecated on RHEL)
- **TOTP 2FA** — optional Google Authenticator integration

## Requirements

- ansible-core >= 2.17

## Supported Platforms

| Platform      | linux-pam | libpwquality | PAM Stack Management |
| ------------- | --------- | ------------ | -------------------- |
| Arch Linux    | 1.7.2     | 1.4.5        | pambase              |
| Debian Trixie | 1.7.0     | 1.4.5        | pam-auth-update      |
| Rocky 9       | 1.5.1     | 1.4.4        | authselect           |
| Rocky 10      | 1.6.1     | 1.4.5        | authselect           |

### Platform-Specific PAM Stack Integration

| Component | Arch Linux             | Debian Trixie               | Rocky 9/10                |
| --------- | ---------------------- | --------------------------- | ------------------------- |
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

## Role Variables

See [`defaults/main.yml`](defaults/main.yml) for all variables with descriptions.

Key toggles:

| Variable                     | Default | Description                          |
| ---------------------------- | ------- | ------------------------------------ |
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
| ------------------------- | ---------------------------- |
| `pam-hardening`           | All role tasks               |
| `pam-hardening:install`   | Package installation         |
| `pam-hardening:configure` | PAM configuration deployment |

## Example Playbook

```yaml
- name: Harden PAM
  hosts: all
  become: true
  tasks:
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

## License

MIT

## Author

Marc Straube
