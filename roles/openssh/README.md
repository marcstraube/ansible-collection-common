# marcstraube.common.openssh

Install and configure OpenSSH server and client with security-hardened defaults.

## Description

This role manages the OpenSSH server (`sshd`) and client configuration using a
modular `sshd_config.d/` drop-in approach. Configuration is split into numbered
snippets for network, host keys, cryptography, authentication, access control,
session, forwarding, subsystems, logging, and match blocks.

Host key types are configurable; algorithm lists (HostKeyAlgorithms,
CASignatureAlgorithms, PubkeyAcceptedAlgorithms) are auto-derived from the
selected key types via an algorithm map, or can be overridden explicitly.

## Requirements

- ansible-core >= 2.17
- Collections: `community.crypto`, `community.general`
- Optional: `ansible.posix` (for firewalld integration)

## Supported Platforms

| Platform                  | Notes                                    |
|---------------------------|------------------------------------------|
| Arch Linux                | OpenSSH 10.3p1, rolling, latest upstream |
| Debian Trixie             | OpenSSH 10.0p1, DSA removed              |
| EL 9 (Rocky, Alma, RHEL)  | OpenSSH 8.7p1, no penalty system         |
| EL 10 (Rocky, Alma, RHEL) | OpenSSH 9.9p1, full feature set          |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

### Version-Conditional Features

| Feature                 | Min Version | Rocky 9 | Rocky 10 | Debian | Arch |
|-------------------------|-------------|---------|----------|--------|------|
| RequiredRSASize         | 9.1         | no      | yes      | yes    | yes  |
| UnusedConnectionTimeout | 9.2         | no      | yes      | yes    | yes  |
| ChannelTimeout          | 9.7         | no      | yes      | yes    | yes  |
| PerSourcePenalties      | 9.8         | no      | yes      | yes    | yes  |
| PrintLastLog            | removed 9.8 | yes     | no       | no     | no   |

## Role Variables

All variables are defined in `defaults/main.yml` with secure defaults.

### Role Control

| Variable                  | Default | Description                          |
|---------------------------|---------|--------------------------------------|
| `openssh_enabled`         | `true`  | Enable the openssh role              |
| `openssh_service_enabled` | `true`  | Enable and start the openssh service |

### Network

| Variable                          | Default | Description                  |
|-----------------------------------|---------|------------------------------|
| `openssh_server_port`             | `22`    | Primary listen port          |
| `openssh_server_additional_ports` | `[]`    | Extra listen ports           |
| `openssh_server_address_family`   | `any`   | `any`, `inet`, `inet6`       |
| `openssh_server_listen_addresses` | `[]`    | Bind addresses (empty = all) |
| `openssh_server_tcp_keep_alive`   | `true`  | TCP keepalive                |

### Cryptography

| Variable                                 | Default          | Description                     |
|------------------------------------------|------------------|---------------------------------|
| `openssh_server_host_key_types`          | `[ed25519, rsa]` | Key types to generate           |
| `openssh_server_rsa_key_size`            | `4096`           | RSA key bits                    |
| `openssh_server_required_rsa_size`       | `3072`           | Min RSA size for clients (9.1+) |
| `openssh_server_kex_algorithms`          | hardened list    | KEX algorithms                  |
| `openssh_server_ciphers`                 | hardened list    | Ciphers                         |
| `openssh_server_macs`                    | hardened list    | MACs                            |
| `openssh_server_host_key_algorithms`     | `auto`           | `auto`, list, or `[]`           |
| `openssh_server_ca_signature_algorithms` | `auto`           | `auto`, list, or `[]`           |

### Authentication

| Variable                                 | Default     | Description       |
|------------------------------------------|-------------|-------------------|
| `openssh_server_authentication_methods`  | `publickey` | Auth methods      |
| `openssh_server_permit_root_login`       | `no`        | Root login policy |
| `openssh_server_password_authentication` | `false`     | Password auth     |
| `openssh_server_pubkey_authentication`   | `true`      | Public key auth   |
| `openssh_server_use_pam`                 | `true`      | PAM integration   |
| `openssh_server_max_auth_tries`          | `3`         | Max auth attempts |

### Access Control

| Variable                      | Default | Description    |
|-------------------------------|---------|----------------|
| `openssh_server_allow_users`  | `[]`    | Allowed users  |
| `openssh_server_deny_users`   | `[]`    | Denied users   |
| `openssh_server_allow_groups` | `[]`    | Allowed groups |
| `openssh_server_deny_groups`  | `[]`    | Denied groups  |

### SFTP Chroot

| Variable                               | Default    | Description                 |
|----------------------------------------|------------|-----------------------------|
| `openssh_server_sftp_enabled`          | `true`     | Enable SFTP subsystem       |
| `openssh_server_sftp_chroot_enabled`   | `false`    | Enable SFTP chroot          |
| `openssh_server_sftp_chroot_directory` | `/home/%u` | Chroot directory            |
| `openssh_server_sftp_chroot_group`     | `sftponly` | Chroot group (auto-created) |

### Moduli Hardening

| Variable                               | Default | Description             |
|----------------------------------------|---------|-------------------------|
| `openssh_server_harden_moduli_enabled` | `true`  | Remove weak DH moduli   |
| `openssh_server_moduli_minimum_size`   | `3071`  | Minimum moduli bit size |

### Penalty System (OpenSSH 9.8+)

| Variable                                   | Default | Description   |
|--------------------------------------------|---------|---------------|
| `openssh_server_per_source_penalties`      | `[]`    | Penalty rules |
| `openssh_server_per_source_net_block_size` | `''`    | CIDR grouping |

### Timeouts (OpenSSH 9.2+/9.7+)

| Variable                                   | Default | Description             |
|--------------------------------------------|---------|-------------------------|
| `openssh_server_unused_connection_timeout` | `''`    | Idle connection timeout |
| `openssh_server_channel_timeout`           | `''`    | Idle channel timeout    |

### Security Tools

| Variable                    | Default | Description              |
|-----------------------------|---------|--------------------------|
| `openssh_ssh_audit_enabled` | `true`  | Enable ssh-audit install |

### Client Configuration

| Variable                                  | Default                    | Description                     |
|-------------------------------------------|----------------------------|---------------------------------|
| `openssh_client_enabled`                  | `true`                     | Enable SSH client configuration |
| `openssh_client_global_known_hosts`       | `/etc/ssh/ssh_known_hosts` | Global known hosts file         |
| `openssh_client_hash_known_hosts`         | `true`                     | Hash known hosts                |
| `openssh_client_strict_host_key_checking` | `ask`                      | Strict host key checking        |

### SSH Agent

| Variable                               | Default | Description                     |
|----------------------------------------|---------|---------------------------------|
| `openssh_agent_global_service_enabled` | `false` | Enable global ssh-agent service |
| `openssh_askpass_enabled`              | `false` | Enable SSH askpass program      |

### Escape Hatch

| Variable                           | Default | Description             |
|------------------------------------|---------|-------------------------|
| `openssh_server_extra_sshd_config` | `[]`    | Extra sshd_config lines |
| `openssh_server_match_blocks`      | `[]`    | Custom Match blocks     |

### Firewall

| Variable                    | Default  | Description            |
|-----------------------------|----------|------------------------|
| `openssh_firewalld_enabled` | `false`  | Enable firewalld rules |
| `openssh_firewalld_zone`    | `public` | Firewalld zone         |

### AppArmor

| Variable                   | Default | Description                         |
|----------------------------|---------|-------------------------------------|
| `openssh_apparmor_enabled` | `false` | Enable AppArmor profile enforcement |

## Tags

| Tag                 | Scope                  |
|---------------------|------------------------|
| `openssh`           | All tasks              |
| `openssh:install`   | Package installation   |
| `openssh:hostkeys`  | Host key generation    |
| `openssh:configure` | Configuration + moduli |
| `openssh:client`    | Client config          |
| `openssh:service`   | Service management     |
| `openssh:firewall`  | Firewalld rules        |
| `openssh:apparmor`  | AppArmor enforcement   |

## Example Playbook

```yaml
- name: Include openssh role
  ansible.builtin.include_role:
    name: marcstraube.common.openssh
  tags:
    - openssh
  when: openssh_enabled | default(true) | bool
```

## Testing

```bash
cd roles/openssh
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

### fail2ban Integration

SSH uses a built-in fail2ban filter. The `marcstraube.common.fail2ban` role
manages the sshd jail centrally via `fail2ban_sshd_enabled`. No fail2ban
configuration is needed in this role.

## License

MIT

## Author

Marc Straube
