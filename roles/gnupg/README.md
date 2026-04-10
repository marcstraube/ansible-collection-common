# marcstraube.common.gnupg

Configure GnuPG for encryption and signing.

## Description

Installs and configures GnuPG with hardened defaults for encryption, signing,
and key management. Manages `gpg.conf`, `gpg-agent.conf`, `scdaemon.conf`, and
`dirmngr.conf` on a per-user basis. Supports SSH agent integration, smartcard
daemon configuration, and keyserver management.

## Requirements

- ansible-core >= 2.17
- No additional collections required

## Supported Platforms

| Platform       | Notes                                     |
| -------------- | ----------------------------------------- |
| Arch Linux     | GnuPG 2.4+, includes GUI tools (Seahorse) |
| Debian Trixie  | GnuPG 2.2+, includes GUI tools (Seahorse) |
| Rocky Linux 9  | GnuPG 2.3+                                |
| Rocky Linux 10 | GnuPG 2.4+                                |

## Role Variables

### Role Control

| Variable        | Default | Description             |
| --------------- | ------- | ----------------------- |
| `gnupg_enabled` | `true`  | Enable/disable the role |

### Packages

| Variable              | Default | Description                                  |
| --------------------- | ------- | -------------------------------------------- |
| `gnupg_install_gui`   | `false` | Install GUI tools (Seahorse, etc.)           |
| `gnupg_install_tools` | `true`  | Install CLI tools (hopenpgp-tools, paperkey) |

### GPG Configuration (gpg.conf)

| Variable                              | Default                       | Description                          |
| ------------------------------------- | ----------------------------- | ------------------------------------ |
| `gnupg_default_key`                   | `''`                          | Default signing key ID               |
| `gnupg_personal_cipher_preferences`   | `AES256 AES192 AES`           | Cipher preferences                   |
| `gnupg_personal_digest_preferences`   | `SHA512 SHA384 SHA256`        | Digest preferences                   |
| `gnupg_personal_compress_preferences` | `ZLIB BZIP2 ZIP Uncompressed` | Compression preferences              |
| `gnupg_default_preference_list`       | *(SHA512, AES256, ...)*       | Default preference list for new keys |
| `gnupg_cert_digest_algo`              | `SHA512`                      | Certificate digest algorithm         |
| `gnupg_trust_model`                   | `tofu+pgp`                    | Trust model                          |
| `gnupg_keyid_format`                  | `0xlong`                      | Key ID display format                |
| `gnupg_display_charset`               | `utf-8`                       | Display character set                |
| `gnupg_no_greeting`                   | `true`                        | Suppress greeting message            |
| `gnupg_no_comments`                   | `true`                        | Strip comments from output           |
| `gnupg_no_emit_version`               | `true`                        | Omit version string                  |
| `gnupg_with_fingerprint`              | `true`                        | Show fingerprints in listings        |
| `gnupg_show_uid_validity`             | `true`                        | Show UID validity                    |
| `gnupg_throw_keyids`                  | `false`                       | Hide recipient key IDs               |
| `gnupg_keyserver`                     | `hkps://keys.openpgp.org`     | Default keyserver                    |
| `gnupg_auto_key_locate`               | `local,wkd`                   | Key locate mechanisms                |

### GPG Agent (gpg-agent.conf)

| Variable                              | Default | Description                           |
| ------------------------------------- | ------- | ------------------------------------- |
| `gnupg_agent_enabled`                 | `true`  | Deploy gpg-agent configuration        |
| `gnupg_agent_default_cache_ttl`       | `600`   | Default cache TTL (seconds)           |
| `gnupg_agent_max_cache_ttl`           | `7200`  | Maximum cache TTL (seconds)           |
| `gnupg_agent_default_cache_ttl_ssh`   | `1800`  | SSH key cache TTL (seconds)           |
| `gnupg_agent_max_cache_ttl_ssh`       | `7200`  | SSH key max cache TTL (seconds)       |
| `gnupg_agent_enable_ssh`              | `false` | Enable SSH support via GPG agent      |
| `gnupg_agent_pinentry`                | `''`    | Pinentry program (empty = OS default) |
| `gnupg_agent_pinentry_timeout`        | `0`     | Pinentry timeout (0 = none)           |
| `gnupg_agent_allow_loopback_pinentry` | `true`  | Allow loopback pinentry               |
| `gnupg_agent_allow_preset_passphrase` | `false` | Allow gpg-preset-passphrase           |
| `gnupg_agent_grab`                    | `false` | Grab keyboard for pinentry (X11)      |
| `gnupg_agent_no_allow_external_cache` | `false` | Disable external passphrase cache     |
| `gnupg_agent_disable_scdaemon`        | `false` | Disable smartcard daemon              |

### Smartcard Daemon (scdaemon.conf)

| Variable                              | Default | Description                                |
| ------------------------------------- | ------- | ------------------------------------------ |
| `gnupg_scdaemon_enabled`              | `true`  | Deploy scdaemon configuration              |
| `gnupg_scdaemon_disable_ccid`         | `false` | Disable integrated CCID driver (use PC/SC) |
| `gnupg_scdaemon_reader_port`          | `''`    | Card reader port (empty = auto-detect)     |
| `gnupg_scdaemon_pcsc_shared`          | `false` | Enable shared PC/SC access                 |
| `gnupg_scdaemon_pcsc_driver`          | `''`    | Custom PC/SC library path                  |
| `gnupg_scdaemon_disable_pinpad`       | `false` | Disable physical PIN pad                   |
| `gnupg_scdaemon_enable_pinpad_varlen` | `false` | Enable variable-length PIN input           |
| `gnupg_scdaemon_deny_admin`           | `false` | Restrict admin card commands               |
| `gnupg_scdaemon_disable_application`  | `[]`    | Disable card applications                  |
| `gnupg_scdaemon_application_priority` | `''`    | Application priority (GnuPG 2.4+)          |
| `gnupg_scdaemon_debug_level`          | `''`    | Debug level                                |
| `gnupg_scdaemon_log_file`             | `''`    | Log file path                              |

### Directory Manager (dirmngr.conf)

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `gnupg_dirmngr_enabled` | `true` | Deploy dirmngr configuration |
| `gnupg_keyservers` | `['hkps://keys.openpgp.org', 'hkps://keyserver.ubuntu.com']` | Keyserver list |
| `gnupg_dirmngr_use_tor` | `false` | Use Tor for keyserver access |
| `gnupg_dirmngr_resolver_timeout` | `30` | DNS resolver timeout (seconds) |
| `gnupg_dirmngr_disable_ipv6` | `false` | Disable IPv6 for keyservers |
| `gnupg_dirmngr_honor_http_proxy` | `false` | Honor HTTP_PROXY env var |
| `gnupg_dirmngr_http_proxy` | `''` | Explicit HTTP proxy |
| `gnupg_dirmngr_allow_ocsp` | `false` | Enable OCSP support |

### Key Management

| Variable            | Default | Description                                     |
| ------------------- | ------- | ----------------------------------------------- |
| `gnupg_import_keys` | `[]`    | Public keys to import (keyid+keyserver or file) |
| `gnupg_trust_keys`  | `[]`    | Key fingerprints to set ultimate trust          |

### User Configuration

| Variable      | Default | Description                                                              |
| ------------- | ------- | ------------------------------------------------------------------------ |
| `gnupg_users` | `[]`    | Users for whom to configure GnuPG (`{username, default_key, agent_ssh}`) |

## Tags

| Tag               | Scope                             |
| ----------------- | --------------------------------- |
| `gnupg`           | All role tasks                    |
| `gnupg:install`   | Package installation              |
| `gnupg:configure` | Per-user configuration deployment |

## Example Playbook

```yaml
- name: Configure GnuPG
  hosts: all
  become: true
  tasks:
    - name: Include gnupg role
      ansible.builtin.include_role:
        name: marcstraube.common.gnupg
      tags: [gnupg]
      when: gnupg_enabled | default(true) | bool
```

### Desktop with SSH Agent and Smartcard

```yaml
- name: Include gnupg role
  ansible.builtin.include_role:
    name: marcstraube.common.gnupg
  vars:
    gnupg_install_gui: true
    gnupg_agent_enable_ssh: true
    gnupg_agent_pinentry: 'pinentry-gnome3'
    gnupg_scdaemon_disable_ccid: true
    gnupg_users:
      - username: 'mst'
        default_key: '0x1234567890ABCDEF'
        agent_ssh: true
```

## Testing

```bash
cd roles/gnupg
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## License

MIT

## Author

Marc Straube
