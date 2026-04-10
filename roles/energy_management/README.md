# marcstraube.common.energy_management

Energy management for desktops and laptops (logind, sleep, PPD/TLP, battery, backlight).

## Description

Manages power-related configuration for both desktops and laptops. Configures systemd
logind (power keys, lid switch, idle action), sleep behavior (suspend, hibernate),
power manager (PPD or TLP), battery charge thresholds, and backlight access. Supports
Arch Linux, Debian Trixie, Rocky Linux 9/10, and Fedora.

## Requirements

- ansible-core >= 2.17
- Collections: `community.general`

## Supported Platforms

| Platform       | Notes                                                               |
| -------------- | ------------------------------------------------------------------- |
| Arch Linux     | Full support including tp_smapi for older ThinkPads                 |
| Debian Trixie  | Full support including tp_smapi (DKMS)                              |
| Rocky Linux 9  | PPD via power-profiles-daemon; no HibernateDelaySec (systemd < 256) |
| Rocky Linux 10 | PPD via tuned + tuned-ppd; tp_smapi not available                   |
| Fedora         | PPD via tuned + tuned-ppd on 41+                                    |

## Role Variables

### Role Control

| Variable                        | Default            | Description             |                                    |
| ------------------------------- | ------------------ | ----------------------- | ---------------------------------- |
| `energy_management_enabled`     | `true`             | Enable/disable the role |                                    |
| `energy_management_system_type` | `"{{ system_type \ | default('desktop') }}"` | System type: `desktop` or `laptop` |

### Logind Configuration

Deployed as `/etc/systemd/logind.conf.d/energy.conf`.

| Variable                                             | Default     | Description                     |
| ---------------------------------------------------- | ----------- | ------------------------------- |
| `energy_management_logind_enabled`                   | `true`      | Deploy logind drop-in           |
| `energy_management_handle_power_key`                 | `suspend`   | Power key action                |
| `energy_management_handle_power_key_long_press`      | `poweroff`  | Long-press power key action     |
| `energy_management_handle_suspend_key`               | `suspend`   | Suspend key action              |
| `energy_management_handle_hibernate_key`             | `hibernate` | Hibernate key action            |
| `energy_management_handle_lid_switch`                | `suspend`   | Lid switch action (laptop only) |
| `energy_management_handle_lid_switch_external_power` | `ignore`    | Lid switch on AC (laptop only)  |
| `energy_management_handle_lid_switch_docked`         | `ignore`    | Lid switch docked (laptop only) |
| `energy_management_idle_action`                      | `''`        | Idle action (empty = disabled)  |
| `energy_management_idle_action_sec`                  | `30min`     | Idle timeout                    |

### Sleep Configuration

Deployed as `/etc/systemd/sleep.conf.d/energy.conf`.

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `energy_management_sleep_enabled` | `true` | Deploy sleep drop-in |
| `energy_management_allow_suspend` | `true` | Allow suspend |
| `energy_management_allow_hibernate` | `true` | Allow hibernate |
| `energy_management_allow_suspend_then_hibernate` | `true` | Allow suspend-then-hibernate |
| `energy_management_allow_hybrid_sleep` | `false` | Allow hybrid sleep |
| `energy_management_suspend_mode` | `''` | Suspend mode (empty = kernel default) |
| `energy_management_suspend_state` | `mem standby freeze` | Suspend state |
| `energy_management_hibernate_mode` | `platform shutdown` | Hibernate mode |
| `energy_management_hibernate_state` | `disk` | Hibernate state |
| `energy_management_hibernate_delay_sec` | `3600` | Suspend-then-hibernate delay (systemd >= 256) |

### Power Manager

| Variable                                | Default    | Description                                 |
| --------------------------------------- | ---------- | ------------------------------------------- |
| `energy_management_power_manager`       | `ppd`      | Power manager: `ppd`, `tlp`, or `''` (none) |
| `energy_management_ppd_default_profile` | `balanced` | Default PPD profile                         |

### TLP Settings

Used when `energy_management_power_manager == 'tlp'`.

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `energy_management_tlp_cpu_energy_perf_policy_on_ac` | `balance_performance` | CPU policy on AC |
| `energy_management_tlp_cpu_energy_perf_policy_on_bat` | `balance_power` | CPU policy on battery |
| `energy_management_tlp_cpu_boost_on_ac` | `1` | CPU boost on AC |
| `energy_management_tlp_cpu_boost_on_bat` | `0` | CPU boost on battery |
| `energy_management_tlp_platform_profile_on_ac` | `performance` | Platform profile on AC |
| `energy_management_tlp_platform_profile_on_bat` | `low-power` | Platform profile on battery |
| `energy_management_tlp_wifi_pwr_on_ac` | `off` | WiFi power saving on AC |
| `energy_management_tlp_wifi_pwr_on_bat` | `on` | WiFi power saving on battery |
| `energy_management_tlp_usb_autosuspend` | `1` | USB autosuspend |
| `energy_management_tlp_runtime_pm_on_ac` | `on` | Runtime PM on AC |
| `energy_management_tlp_runtime_pm_on_bat` | `auto` | Runtime PM on battery |
| `energy_management_tlp_nmi_watchdog` | `0` | NMI watchdog (0 = disabled) |
| `energy_management_tlp_extra_settings` | `{}` | Additional raw key=value pairs for tlp.conf |

### Battery Threshold (Laptop Only)

| Variable                                    | Default | Description                      |
| ------------------------------------------- | ------- | -------------------------------- |
| `energy_management_battery_enabled`         | `false` | Enable battery charge thresholds |
| `energy_management_battery_start_threshold` | `75`    | Start charging threshold (%)     |
| `energy_management_battery_stop_threshold`  | `80`    | Stop charging threshold (%)      |
| `energy_management_battery_tp_smapi`        | `false` | Use tp_smapi for older ThinkPads |

### Backlight (Laptop Only)

| Variable                              | Default | Description                          |
| ------------------------------------- | ------- | ------------------------------------ |
| `energy_management_backlight_enabled` | `false` | Install brightnessctl and udev rules |

## Tags

| Tag                | Scope                       |
| ------------------ | --------------------------- |
| `energy`           | All role tasks              |
| `energy:logind`    | Logind configuration        |
| `energy:sleep`     | Sleep configuration         |
| `energy:ppd`       | Power-profiles-daemon tasks |
| `energy:tlp`       | TLP tasks                   |
| `energy:battery`   | Battery threshold tasks     |
| `energy:backlight` | Backlight tasks             |

## Example Playbook

```yaml
- name: Configure energy management
  hosts: all
  become: true
  tasks:
    - name: Include energy_management role
      ansible.builtin.include_role:
        name: marcstraube.common.energy_management
      tags: [energy]
      when: energy_management_enabled | default(true) | bool
```

### Laptop with TLP and Battery Thresholds

```yaml
- name: Include energy_management role
  ansible.builtin.include_role:
    name: marcstraube.common.energy_management
  vars:
    energy_management_system_type: 'laptop'
    energy_management_power_manager: 'tlp'
    energy_management_battery_enabled: true
    energy_management_battery_start_threshold: 75
    energy_management_battery_stop_threshold: 80
    energy_management_backlight_enabled: true
```

## Testing

```bash
cd roles/energy_management
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## License

MIT

## Author

Marc Straube
