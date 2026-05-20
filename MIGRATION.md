# Migration Guide

Upgrade guide for users of `marcstraube.common`. Each release with
breaking changes adds a section here with before/after inventory
snippets. For the full list of changes per release, see
[CHANGELOG.md](CHANGELOG.md).

## v2.0.0 (unreleased)

> **Note**: this section is in progress. Earlier `v2.0.0` breaking
> changes are tracked in the v2.0.0 migration backfill issue and will
> be added before the release tag.

### `networkmanager` — generic `settings:` dict (#153)

Three per-connection fields are removed and must be migrated into the
new generic `settings:` dict, which takes nmcli-native keys verbatim.

#### Field renames

| Old per-connection field         | New `settings:` key                                |
|----------------------------------|----------------------------------------------------|
| `mac: "..."`                     | `802-11-wireless.cloned-mac-address: "..."` (WiFi) |
| `dhcp_send_hostname_ipv4: true`  | `ipv4.dhcp-send-hostname: "yes"`                   |
| `dhcp_send_hostname_ipv4: false` | `ipv4.dhcp-send-hostname: "no"`                    |
| `dhcp_send_hostname_ipv6: true`  | `ipv6.dhcp-send-hostname: "yes"`                   |
| `dhcp_send_hostname_ipv6: false` | `ipv6.dhcp-send-hostname: "no"`                    |

Note: the old `mac:` field was overloaded — it was passed to the
`community.general.nmcli` module as the device MAC selector *and* used
for the cloned MAC address. The new schema separates these:

- `802-11-wireless.cloned-mac-address` — clone / randomize on connect
- `802-11-wireless.mac-address` — device selector (lock connection to this MAC)
- `802-3-ethernet.mac-address` — ethernet device selector

#### Before

```yaml
networkmanager_connections:
  ethernet:
    "Office":
      ifname: "eth0"
      dhcp_send_hostname_ipv4: false
      dhcp_send_hostname_ipv6: false
  wifi:
    "Home WiFi":
      ssid: "MyNetwork"
      mac: "random"
```

#### After

```yaml
networkmanager_connections:
  ethernet:
    "Office":
      ifname: "eth0"
      settings:
        ipv4.dhcp-send-hostname: "no"
        ipv6.dhcp-send-hostname: "no"
  wifi:
    "Home WiFi":
      ssid: "MyNetwork"
      settings:
        802-11-wireless.cloned-mac-address: "random"
```

#### Value formats

For ternary-boolean keys (`*.dhcp-send-hostname`, `connection.metered`,
etc.) any of nmcli's accepted forms work: `yes`/`true`/`1`,
`no`/`false`/`0`, or `default`/`unknown`/`-1`. The role normalizes both
sides before comparing, so the choice is purely cosmetic.

For all other keys (strings, MAC addresses, enums like `random`), use
the value verbatim — comparison is case-sensitive against the exact
string nmcli returns.

The global daemon-config defaults (`networkmanager_dhcp_send_hostname_ipv4`
/ `_ipv6`) are unaffected by this change — they continue to write the
`[connection]` section of `/etc/NetworkManager/conf.d/00-ansible.conf`.
