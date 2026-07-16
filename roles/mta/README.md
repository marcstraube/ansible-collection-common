# mta

Configures **Postfix** as a lightweight, send-only system MTA (null client) so
that local processes — cron jobs, backup scripts, `clamav`, `fail2ban`, `aide`
and anything else calling `/usr/sbin/sendmail` — can deliver mail.

By default Postfix delivers directly via DNS/MX lookup (exactly like a stock
server install). Set `mta_relayhost` to route everything through a smarthost
instead. The host binds to loopback only and accepts no inbound mail.

This role intentionally stays a **null client**. For a full inbound mail server
(MX, spam filtering, authenticated submission, virtual domains) use the
`marcstraube.server.postfix` role instead — do not enable both on one host.

## Requirements

- `community.general` collection (for the `alternatives` module)
- A resolvable hostname/domain for a sensible envelope origin (override
  `mta_myhostname` / `mta_mydomain` if the facts are not usable)

## Supported Platforms

| Platform                  | Notes                                           |
|---------------------------|-------------------------------------------------|
| Arch Linux                | Postfix provides `/usr/bin/sendmail` directly   |
| Debian Trixie             | Installed non-interactively via debconf preseed |
| EL 9 (Rocky, Alma, RHEL)  | Postfix selected as the `mta` alternative       |
| EL 10 (Rocky, Alma, RHEL) | Postfix selected as the `mta` alternative       |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable              | Default | Description                          |
|-----------------------|---------|--------------------------------------|
| `mta_enabled`         | `true`  | Enable the mta role                  |
| `mta_service_enabled` | `true`  | Enable and start the Postfix service |

### Identity

| Variable         | Default                       | Description                                            |
|------------------|-------------------------------|--------------------------------------------------------|
| `mta_myhostname` | `{{ ansible_facts['fqdn'] }}` | Envelope origin host (`myhostname`)                    |
| `mta_mydomain`   | host domain / `localdomain`   | Domain for unqualified addresses (`mydomain`)          |
| `mta_myorigin`   | `{{ mta_mydomain }}`          | Domain outgoing mail appears to come from (`myorigin`) |

### Mail Routing

| Variable              | Default                   | Description                                            |
|-----------------------|---------------------------|--------------------------------------------------------|
| `mta_relayhost`       | `''`                      | Smarthost to relay through. Empty = direct MX delivery |
| `mta_mynetworks`      | `127.0.0.0/8 [::1]/128`   | Networks permitted to relay (`mynetworks`)             |
| `mta_inet_interfaces` | `loopback-only`           | Interfaces Postfix binds to (`inet_interfaces`)        |
| `mta_inet_protocols`  | `all`                     | IP protocols (`inet_protocols`)                        |

### Extra Configuration

| Variable               | Default | Description                                                  |
|------------------------|---------|--------------------------------------------------------------|
| `mta_extra_parameters` | `{}`    | Additional `main.cf` `key: value` pairs applied via postconf |

## Tags

| Tag             | Effect                                |
|-----------------|---------------------------------------|
| `mta`           | Everything in the role                |
| `mta:install`   | Package installation only             |
| `mta:configure` | `main.cf` parameters + MTA selection  |
| `mta:service`   | Service management only               |

## Example Playbook

Direct delivery (default — like a stock server):

```yaml
- hosts: servers
  become: true
  roles:
    - role: marcstraube.common.mta
```

Relay everything through a smarthost:

```yaml
- hosts: servers
  become: true
  roles:
    - role: marcstraube.common.mta
      vars:
        mta_relayhost: '[smtp.example.com]:587'
```

## Testing

```bash
cd roles/mta
molecule test
```

Molecule tests package installation, `main.cf` parameters, MTA selection,
service state and a functional `sendmail` submission across Arch Linux, Debian
Trixie and Rocky Linux 9/10.

## Notes

- `main.cf` is edited key-by-key with `postconf`, not overwritten with a
  template, so the distribution's compiled-in defaults (queue paths, owners,
  alias maps) stay intact and only the parameters this role owns change.
- On a host that should run the full mail stack, disable this role
  (`mta_enabled: false`) and use `marcstraube.server.postfix` instead.

## References

- [Postfix documentation](https://www.postfix.org/documentation.html)
- [Postfix null client](https://www.postfix.org/STANDARD_CONFIGURATION_README.html#null_client)
- [postconf(5)](https://www.postfix.org/postconf.5.html)

## License

MIT

## Author

Marc Straube
