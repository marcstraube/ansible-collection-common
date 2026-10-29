# Ansible Role: wazuh_agent

Install the Wazuh agent and enroll it with a Wazuh manager.

The role installs a pinned agent release, deploys `ossec.conf` and lets the
agent enroll on its first start, optionally with a password and
certificates. The manager side is
[marcstraube.server.wazuh_server](https://github.com/marcstraube/ansible-collection-server/tree/develop/roles/wazuh_server).

## Requirements

- ansible-core >= 2.19
- A Wazuh manager of the same or a newer version than `wazuh_agent_version`
- Arch Linux: an AUR build user (`aur_builder`, see `aur_builder_user`)

### Required Collections

```yaml
collections:
  - community.general
  - kewlfft.aur
```

## Supported Platforms

| Platform                  | Notes                                   |
| ------------------------- | --------------------------------------- |
| Arch Linux                | AUR package `wazuh-agent`               |
| Debian Trixie             | Wazuh repository                        |
| EL 9 (Rocky, Alma, RHEL)  | Wazuh repository                        |
| EL 10 (Rocky, Alma, RHEL) | Wazuh repository                        |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu,
Mint, Fedora) should work but are not actively tested. Use
distro-specific vars overrides if needed.

## Role Variables

### Role Control

| Variable                      | Default | Description                         |
| ----------------------------- | ------- | ----------------------------------- |
| `wazuh_agent_enabled`         | `false` | Enable the wazuh_agent role         |
| `wazuh_agent_service_enabled` | `true`  | Enable and start the agent service  |

### Installation

| Variable              | Default    | Description         |
| --------------------- | ---------- | ------------------- |
| `wazuh_agent_version` | `'4.14.9'` | Wazuh agent release |

The agent must not be newer than the manager. Update the manager first,
then raise `wazuh_agent_version`:

- Debian and EL install exactly this version, so a raised version updates
  the agent. A version lower than the installed one fails the run. The
  package is held at this version (`dpkg` hold on Debian, `dnf versionlock`
  on EL), so a system upgrade leaves the agent alone. A raised version
  releases the hold for the update and sets it again.
- Arch Linux installs the AUR package, which always builds the current
  release. The role checks the release the AUR holds on every run and
  installs or updates the agent only while that release is not newer than
  `wazuh_agent_version`. A newer AUR release fails a first install and
  leaves an installed agent alone. The run also fails when the installed
  agent is newer than `wazuh_agent_version`.
- On Arch Linux the role places `IgnorePkg = wazuh-agent` in
  `/etc/pacman.d/ignore.d/wazuh-agent.conf`, so that a system upgrade with
  pacman or an AUR helper leaves the agent alone. pacman reads the file
  only when the `[options]` section of `/etc/pacman.conf` contains
  `Include = /etc/pacman.d/ignore.d/*.conf`, which
  `marcstraube.common.package_management` sets up. Without that role, add
  the line yourself, or add `wazuh-agent` to `IgnorePkg` in
  `/etc/pacman.conf`. pacman fails on an `Include` glob that matches no
  file.

### Manager Connection

| Variable               | Default | Description                              |
| ---------------------- | ------- | ---------------------------------------- |
| `wazuh_agent_managers` | `[]`    | Managers the agent reports to (required) |

Each entry takes `address`, `port` (default `1514`) and `protocol` (`tcp`
or `udp`, default `tcp`). The agent connects to the first manager and fails
over to the next.

### Enrollment

| Variable                              | Default                      | Description                                   |
| ------------------------------------- | ---------------------------- | --------------------------------------------- |
| `wazuh_agent_enrollment_enabled`      | `true`                       | Enroll the agent automatically                |
| `wazuh_agent_enrollment_port`         | `1515`                       | Port of the enrollment service on the manager |
| `wazuh_agent_name`                    | `'{{ inventory_hostname }}'` | Name the agent enrolls with                   |
| `wazuh_agent_groups`                  | `[]`                         | Agent groups assigned on enrollment           |
| `wazuh_agent_enrollment_password`     | `''`                         | Enrollment password, empty for none           |
| `wazuh_agent_enrollment_ca_cert_file` | `''`                         | CA certificate to verify the manager          |
| `wazuh_agent_enrollment_cert_file`    | `''`                         | Agent certificate for enrollment              |
| `wazuh_agent_enrollment_key_file`     | `''`                         | Agent key for enrollment                      |

Keep the password in an Ansible Vault. Set the agent certificate and key
together or not at all. See [Enrollment](#enrollment-1).

### Configuration

| Variable                           | Default | Description                                   |
| ---------------------------------- | ------- | --------------------------------------------- |
| `wazuh_agent_localfiles`           | `[]`    | Additional log sources (`<localfile>`)        |
| `wazuh_agent_syscheck_directories` | `[]`    | Additional directories for integrity checks   |

Each `wazuh_agent_localfiles` entry is a mapping whose keys become elements
of a `<localfile>` block. Each `wazuh_agent_syscheck_directories` entry
takes a `path` and optional `attributes` for the `<directories>` element:

```yaml
wazuh_agent_localfiles:
  - log_format: 'syslog'
    location: '/var/log/nginx/error.log'

wazuh_agent_syscheck_directories:
  - path: '/srv/www'
    attributes:
      realtime: 'yes'
      report_changes: 'yes'
```

## Tags

| Tag                     | Description                          |
| ----------------------- | ------------------------------------ |
| `wazuh-agent`           | All wazuh_agent tasks                |
| `wazuh-agent:install`   | Repository and package installation  |
| `wazuh-agent:certs`     | Enrollment certificate deployment    |
| `wazuh-agent:configure` | `ossec.conf` and enrollment password |
| `wazuh-agent:service`   | Systemd service management           |

## Example Playbook

```yaml
- hosts: servers
  become: true
  vars:
    pki_ca_enabled: true
    pki_client_certs:
      - name: 'wazuh-agent'
        common_name: '{{ inventory_hostname }}'

    wazuh_agent_enabled: true
    wazuh_agent_managers:
      - address: 'wazuh.example.com'
    wazuh_agent_groups:
      - 'linux'
    wazuh_agent_enrollment_password: '{{ vault_wazuh_agent_enrollment_password }}'
    wazuh_agent_enrollment_ca_cert_file: '/etc/ssl/ca/certs/ca.crt'
    wazuh_agent_enrollment_cert_file: '/etc/ssl/certs/wazuh-agent.crt'
    wazuh_agent_enrollment_key_file: '/etc/ssl/private/wazuh-agent.key'
  roles:
    - marcstraube.common.pki
    - marcstraube.common.wazuh_agent
```

The paths above are the pki defaults (`pki_ca_dir`, `pki_certs_dir`,
`pki_private_dir`).

## Testing

```bash
# Run full test (one platform at a time)
molecule test -p wazuh-agent-rocky-10

# Converge only
molecule converge -p wazuh-agent-rocky-10
```

Driver: `podman` | Scenario: `default` (no manager; certificates issued by
`marcstraube.common.pki` in `prepare.yml`).

## Notes

### Configuration

`ossec.conf` follows the upstream agent default of the installed release and
is the same on every platform. The package log of the platform
(`/var/log/dpkg.log`, `/var/log/pacman.log`) is collected in addition to
the journal. The configuration profiles (`<config-profile>`) match those the
package installer writes, so `agent.conf` on the manager can address them.
Tune modules per agent group in the centralized `agent.conf` on the manager
instead of in this role.

### Enrollment

The agent enrolls on its first start and stores its key in `client.keys`;
the role does not touch that file. With `wazuh_agent_enrollment_password`
set, the role writes it to `/var/ossec/etc/authd.pass` (`0640 root:wazuh`)
and removes the file when the password is emptied. The enrollment
certificates are copied into `/var/ossec/etc`, because the agent daemons
cannot read keys that only root may read. A changed source file restarts
the agent.

### Firewall

The agent only opens outgoing connections to the manager: `1514/tcp` for
events and `1515/tcp` for enrollment. No incoming ports are needed. Open
them on the manager through the `marcstraube.common.firewalld` inventory.

## References

- [Wazuh agent installation](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html)
- [Wazuh agent enrollment](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/index.html)
- [Local configuration (ossec.conf)](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/index.html)
- [Centralized configuration (agent.conf)](https://documentation.wazuh.com/current/user-manual/reference/centralized-configuration.html)

## License

MIT

## Author

Marc Straube
