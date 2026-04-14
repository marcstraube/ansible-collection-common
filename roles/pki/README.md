# marcstraube.common.pki

PKI -- certificate and key infrastructure management.

## Description

Manages OpenSSL configuration, Certificate Authorities (root + intermediate),
server/client certificates, CSRs, DH parameters, CRL, OCSP responder, and certificate
expiry monitoring.

## Requirements

- ansible-core >= 2.17
- Collections: `community.crypto`, `ansible.posix`
- Python `cryptography` library on target hosts

## Supported Platforms

| Platform                   | Notes |
|----------------------------|-------|
| Arch Linux                 |       |
| Debian Trixie              |       |
| EL 9 (Rocky, Alma, RHEL)  |       |
| EL 10 (Rocky, Alma, RHEL) |       |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_enabled` | `true` | Enable the pki role |

### OpenSSL System Configuration

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_system_config_enabled` | `true` | Enable system-wide OpenSSL configuration |
| `pki_min_protocol` | `'TLSv1.2'` | Minimum TLS version |
| `pki_max_protocol` | `''` | Maximum TLS version (empty = no limit) |
| `pki_ciphersuites_tls12` | `'ECDHE+AESGCM:...'` | TLS 1.2 cipher suites |
| `pki_ciphersuites_tls13` | `'TLS_AES_256_GCM_SHA384:...'` | TLS 1.3 cipher suites |
| `pki_disable_legacy_enabled` | `true` | Enable disabling of legacy/insecure algorithms |
| `pki_crypto_policy` | `'DEFAULT'` | RHEL/Rocky crypto-policy |

On RHEL/Rocky, system-wide TLS is managed via `update-crypto-policies`. On Arch/Debian,
a drop-in OpenSSL config is deployed.

### Certificate Authority

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_ca_enabled` | `false` | Enable local Certificate Authority |
| `pki_ca_dir` | `'/etc/ssl/ca'` | CA base directory |
| `pki_ca_key_type` | `'ec'` | Key type: `rsa`, `ec`, `ed25519` |
| `pki_ca_key_size` | `4096` | RSA key size |
| `pki_ca_key_curve` | `'secp384r1'` | EC curve |
| `pki_ca_validity_days` | `3650` | CA cert validity (10 years) |
| `pki_ca_country` | `''` | CA subject: country |
| `pki_ca_state` | `''` | CA subject: state |
| `pki_ca_locality` | `''` | CA subject: locality |
| `pki_ca_organization` | `'Private CA'` | CA subject: organization |
| `pki_ca_common_name` | `'Private Root CA'` | CA subject: common name |
| `pki_ca_passphrase` | vault reference | CA key passphrase (use vault!) |

### Intermediate CA

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_intermediate_ca_enabled` | `false` | Enable intermediate CA |
| `pki_intermediate_ca_dir` | `'/etc/ssl/intermediate'` | Intermediate CA directory |
| `pki_intermediate_ca_key_type` | `'ec'` | Key type |
| `pki_intermediate_ca_key_curve` | `'secp384r1'` | EC curve |
| `pki_intermediate_ca_validity_days` | `1825` | Validity (5 years) |
| `pki_intermediate_ca_passphrase` | vault reference | Key passphrase |

### Server Certificates

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_server_certs` | `[]` | List of server certificates to generate |
| `pki_server_key_type` | `'ec'` | Default key type |
| `pki_server_key_size` | `2048` | Default RSA key size |
| `pki_server_key_curve` | `'secp256r1'` | Default EC curve |
| `pki_server_validity_days` | `365` | Default validity |
| `pki_server_key_mode` | `'0600'` | Default key file permissions |

Server cert dict keys: `name`, `common_name`, `san`, `key_type`, `key_curve`,
`validity_days`, `dest_cert`, `dest_key`, `dest_chain`, `owner`, `group`, `key_mode`,
`sign_with_ca`.

### Client Certificates

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_client_certs` | `[]` | List of client certificates to generate |
| `pki_client_key_type` | `'ec'` | Default key type |
| `pki_client_key_curve` | `'secp256r1'` | Default EC curve |
| `pki_client_validity_days` | `365` | Default validity |

Client cert dict keys: `name`, `common_name`, `email`, `validity_days`,
`export_pkcs12`, `pkcs12_passphrase`.

### DH Parameters

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_dhparam_enabled` | `true` | Enable DH parameter generation |
| `pki_dhparam_size` | `4096` | DH param size (2048 or 4096) |
| `pki_dhparam_path` | `'/etc/ssl/dhparam.pem'` | Output file path |

### Certificate Directories

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_certs_dir` | `'/etc/ssl/certs'` | Certificate directory |
| `pki_private_dir` | `'/etc/ssl/private'` | Private key directory |
| `pki_csr_dir` | `'/etc/ssl/csr'` | CSR directory |
| `pki_create_dirs_enabled` | `true` | Enable directory creation |
| `pki_private_dir_mode` | `'0700'` | Private directory permissions |

### Trusted CAs

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_trusted_cas` | `[]` | Additional CA certs to trust system-wide |

Trusted CA dict keys: `name`, `cert` (inline content) or `src` (file path).

### Certificate Deployment

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_deploy_certs` | `[]` | Group-level certs from inventory `files/certs/` |
| `pki_deploy_certs_host` | `[]` | Host-level certs (additive) |

### CRL

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_crl_enabled` | `false` | Enable CRL management |
| `pki_crl_validity_days` | `30` | CRL validity |
| `pki_revoked_certs` | `[]` | List of revoked certificates |

### OCSP

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_ocsp_enabled` | `false` | Enable OCSP responder |
| `pki_ocsp_url` | `''` | OCSP responder URL |
| `pki_ocsp_port` | `2560` | OCSP responder listen port |

### Renewal Monitoring

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_renewal_enabled` | `false` | Enable certificate expiry monitoring |
| `pki_renewal_warning_days` | `30` | Warning threshold (days) |
| `pki_renewal_timer_oncalendar` | `'weekly'` | Timer schedule |
| `pki_renewal_report_email` | `'root@localhost'` | Email for notifications |

### Firewall

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_firewalld_enabled` | `false` | Enable firewalld rule for OCSP port |
| `pki_firewalld_zone` | `'public'` | Firewalld zone for OCSP responder |

### AppArmor

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `pki_apparmor_enabled` | `false` | Enable AppArmor profile for OCSP |

## Tags

| Tag | Scope |
| --- | ----- |
| `pki` | All PKI tasks |
| `pki:install` | Package installation |
| `pki:configure` | System config, directories, trusted CAs |
| `pki:ca` | Root CA, intermediate CA, CRL |
| `pki:certs` | Server/client certs, CSRs, deployment |
| `pki:dhparam` | DH parameter generation |
| `pki:ocsp` | OCSP responder |
| `pki:renewal` | Expiry monitoring |
| `pki:firewall` | Firewalld rules |
| `pki:apparmor` | AppArmor profile |

## Example Playbook

```yaml
- name: Include pki role
  ansible.builtin.include_role:
    name: marcstraube.common.pki
  tags:
    - pki
  when: pki_enabled | default(true) | bool
```

## Example Inventory

```yaml
# group_vars/all/vars.yml
pki_enabled: true
pki_dhparam_enabled: true
pki_dhparam_size: 4096

# group_vars/web_servers/vars.yml
pki_ca_enabled: true
pki_intermediate_ca_enabled: true
pki_server_certs:
  - name: 'webserver'
    common_name: 'www.example.com'
    san:
      - 'DNS:www.example.com'
      - 'DNS:example.com'
    dest_cert: '/etc/ssl/certs/webserver.crt'
    dest_key: '/etc/ssl/private/webserver.key'
    dest_chain: '/etc/ssl/certs/webserver-chain.crt'
    group: 'ssl-cert'
    key_mode: '0640'
```

## Testing

```bash
cd roles/pki
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## License

MIT

## Author

Marc Straube
