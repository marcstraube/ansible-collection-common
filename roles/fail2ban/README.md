# marcstraube.common.fail2ban

Intrusion prevention with Fail2ban.

## Description

Installs and configures Fail2ban for intrusion prevention. Provides pre-configured
jails for SSH, mail servers (Postfix, Dovecot, SOGo, Roundcube), web servers
(Nginx, Apache), databases (MariaDB, PostgreSQL), and monitoring (Grafana).
Supports progressive ban times, recidive detection, custom jails/filters/actions,
and optional AppArmor confinement.

## Requirements

- ansible-core >= 2.17
- No additional collections required

## Supported Platforms

| Platform       | Notes                            |
| -------------- | -------------------------------- |
| Arch Linux     | Ban action defaults to firewalld |
| Debian Trixie  | Ban action defaults to firewalld |
| Rocky Linux 9  | Ban action defaults to firewalld |
| Rocky Linux 10 | Ban action defaults to firewalld |

## Role Variables

### Service Control

| Variable           | Default | Description                     |
| ------------------ | ------- | ------------------------------- |
| `fail2ban_enabled` | `true`  | Enable/disable fail2ban service |

### Global Settings

| Variable              | Default                  | Description                    |
| --------------------- | ------------------------ | ------------------------------ |
| `fail2ban_ignoreip`   | `['127.0.0.1/8', '::1']` | IPs to never ban               |
| `fail2ban_ignoreself` | `true`                   | Ignore own IP addresses        |
| `fail2ban_bantime`    | `1h`                     | Default ban duration           |
| `fail2ban_findtime`   | `10m`                    | Time window for maxretry       |
| `fail2ban_maxretry`   | `5`                      | Default max retries before ban |

### Progressive Ban Times

| Variable                        | Default         | Description                  |
| ------------------------------- | --------------- | ---------------------------- |
| `fail2ban_bantime_increment`    | `false`         | Enable progressive ban times |
| `fail2ban_bantime_factor`       | `2`             | Ban time multiplier          |
| `fail2ban_bantime_formula`      | *(exponential)* | Ban time calculation formula |
| `fail2ban_bantime_maxtime`      | `5w`            | Maximum ban duration         |
| `fail2ban_bantime_overalljails` | `false`         | Count bans across all jails  |

### Ban Action

| Variable                      | Default         | Description             |
| ----------------------------- | --------------- | ----------------------- |
| `fail2ban_banaction`          | *(OS-specific)* | Ban action backend      |
| `fail2ban_banaction_allports` | *(OS-specific)* | All-ports ban action    |
| `fail2ban_chain`              | `INPUT`         | iptables/nftables chain |

### Logging

| Variable                 | Default                 | Description   |
| ------------------------ | ----------------------- | ------------- |
| `fail2ban_loglevel`      | `INFO`                  | Log level     |
| `fail2ban_logtarget`     | `/var/log/fail2ban.log` | Log target    |
| `fail2ban_syslog_target` | `/dev/log`              | Syslog target |

### Mail Notifications

| Variable              | Default              | Description                                  |
| --------------------- | -------------------- | -------------------------------------------- |
| `fail2ban_destemail`  | `root@localhost`     | Notification recipient                       |
| `fail2ban_sender`     | `fail2ban@localhost` | Sender address                               |
| `fail2ban_sendername` | `Fail2Ban`           | Sender display name                          |
| `fail2ban_mta`        | `sendmail`           | Mail transfer agent                          |
| `fail2ban_action`     | `%(action_)s`        | Action type (ban only, +mail, +whois, +logs) |

### Database

| Variable                | Default                              | Description                   |
| ----------------------- | ------------------------------------ | ----------------------------- |
| `fail2ban_dbfile`       | `/var/lib/fail2ban/fail2ban.sqlite3` | Database file path            |
| `fail2ban_dbpurgeage`   | `1d`                                 | Database purge age            |
| `fail2ban_dbmaxmatches` | `10`                                 | Max matches stored per ticket |

### Default Jail Settings

| Variable                       | Default | Description       |
| ------------------------------ | ------- | ----------------- |
| `fail2ban_default_backend`     | `auto`  | Detection backend |
| `fail2ban_default_usedns`      | `warn`  | DNS usage policy  |
| `fail2ban_default_logencoding` | `auto`  | Log file encoding |

### SSH Jails

| Variable                      | Default  | Description                                 |
| ----------------------------- | -------- | ------------------------------------------- |
| `fail2ban_sshd_enabled`       | `true`   | Enable SSH jail                             |
| `fail2ban_sshd_port`          | `ssh`    | SSH port                                    |
| `fail2ban_sshd_maxretry`      | `5`      | Max retries                                 |
| `fail2ban_sshd_bantime`       | `1h`     | Ban duration                                |
| `fail2ban_sshd_mode`          | `normal` | Filter mode: `normal`, `ddos`, `aggressive` |
| `fail2ban_sshd_ddos_enabled`  | `false`  | Enable SSH DDoS jail                        |
| `fail2ban_sshd_ddos_maxretry` | `6`      | DDoS max retries                            |

### Mail Server Jails

| Variable                        | Default | Description              |
| ------------------------------- | ------- | ------------------------ |
| `fail2ban_postfix_enabled`      | `false` | Enable Postfix jail      |
| `fail2ban_postfix_sasl_enabled` | `false` | Enable Postfix SASL jail |
| `fail2ban_postfix_rbl_enabled`  | `false` | Enable Postfix RBL jail  |
| `fail2ban_dovecot_enabled`      | `false` | Enable Dovecot jail      |
| `fail2ban_sogo_enabled`         | `false` | Enable SOGo jail         |
| `fail2ban_roundcube_enabled`    | `false` | Enable Roundcube jail    |

### Web Server Jails

| Variable                             | Default | Description                   |
| ------------------------------------ | ------- | ----------------------------- |
| `fail2ban_nginx_http_auth_enabled`   | `false` | Enable Nginx HTTP auth jail   |
| `fail2ban_nginx_limit_req_enabled`   | `false` | Enable Nginx limit-req jail   |
| `fail2ban_nginx_botsearch_enabled`   | `false` | Enable Nginx bot search jail  |
| `fail2ban_nginx_forbidden_enabled`   | `false` | Enable Nginx forbidden jail   |
| `fail2ban_nginx_bad_request_enabled` | `false` | Enable Nginx bad request jail |
| `fail2ban_apache_auth_enabled`       | `false` | Enable Apache auth jail       |
| `fail2ban_apache_badbots_enabled`    | `false` | Enable Apache bad bots jail   |

### Database Jails

| Variable                      | Default | Description               |
| ----------------------------- | ------- | ------------------------- |
| `fail2ban_mysqld_enabled`     | `false` | Enable MariaDB/MySQL jail |
| `fail2ban_postgresql_enabled` | `false` | Enable PostgreSQL jail    |

### Monitoring Jails

| Variable                   | Default | Description         |
| -------------------------- | ------- | ------------------- |
| `fail2ban_grafana_enabled` | `false` | Enable Grafana jail |

### Recidive (Repeat Offenders)

| Variable                     | Default | Description                   |
| ---------------------------- | ------- | ----------------------------- |
| `fail2ban_recidive_enabled`  | `false` | Enable recidive jail          |
| `fail2ban_recidive_bantime`  | `1w`    | Recidive ban duration         |
| `fail2ban_recidive_findtime` | `1d`    | Recidive detection window     |
| `fail2ban_recidive_maxretry` | `5`     | Bans before recidive triggers |

### Custom Jails, Filters, and Actions

| Variable                  | Default | Description                       |
| ------------------------- | ------- | --------------------------------- |
| `fail2ban_custom_jails`   | `[]`    | List of custom jail definitions   |
| `fail2ban_custom_filters` | `[]`    | List of custom filter definitions |
| `fail2ban_custom_actions` | `[]`    | List of custom action definitions |

### AppArmor

| Variable                    | Default | Description                                 |
| --------------------------- | ------- | ------------------------------------------- |
| `fail2ban_apparmor_enabled` | `false` | Enforce AppArmor profile (Arch/Debian only) |

## Tags

| Tag                  | Scope                         |
| -------------------- | ----------------------------- |
| `fail2ban`           | All role tasks                |
| `fail2ban:install`   | Package installation          |
| `fail2ban:configure` | Configuration and BTRFS tasks |
| `fail2ban:service`   | Service management            |
| `fail2ban:apparmor`  | AppArmor profile enforcement  |

## Example Playbook

```yaml
- name: Configure fail2ban
  hosts: all
  become: true
  tasks:
    - name: Include fail2ban role
      ansible.builtin.include_role:
        name: marcstraube.common.fail2ban
      tags: [fail2ban]
      when: fail2ban_enabled | default(true) | bool
```

### Mail Server with Progressive Bans

```yaml
- name: Include fail2ban role
  ansible.builtin.include_role:
    name: marcstraube.common.fail2ban
  vars:
    fail2ban_bantime_increment: true
    fail2ban_postfix_enabled: true
    fail2ban_postfix_sasl_enabled: true
    fail2ban_dovecot_enabled: true
    fail2ban_recidive_enabled: true
```

## Testing

```bash
cd roles/fail2ban
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## License

MIT

## Author

Marc Straube
