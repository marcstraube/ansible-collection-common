# marcstraube.common.utils

Install common system utilities: process monitors, file transfer tools, file managers,
system info fetchers, and fun terminal toys.

## Description

This is an install-only role with no configuration or services. Each utility is gated by a
boolean toggle and mapped to OS-specific package names. Unavailable packages (empty string
in vars) are silently skipped.

Terminal file managers (mc, vifm, ranger, nnn, lf) are managed here since they are useful
on all host types including servers. For GUI file managers (Thunar, Dolphin, Nautilus, etc.),
see `marcstraube.desktop.filemanagers`.

## Requirements

- `kewlfft.aur` collection (Arch Linux AUR packages only)
- EPEL repository enabled on Rocky/RHEL (for btop, cowsay, figlet, etc.)

## Supported Platforms

| Platform                   | Notes         |
|----------------------------|---------------|
| Arch Linux                 |               |
| Debian Trixie              |               |
| EL 9 (Rocky, Alma, RHEL)   | EPEL required |
| EL 10 (Rocky, Alma, RHEL)  | EPEL required |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

Package availability varies by platform. See `defaults/main.yml` comments and `vars/` files for details.

## Role Variables

All variables default to `false` unless noted. See `defaults/main.yml` for the complete
list with availability notes per OS.

### Role Control

| Variable        | Default | Description           |
|-----------------|---------|-----------------------|
| `utils_enabled` | `true`  | Enable the utils role |

### Monitors

| Variable                       | Default | Description                                |
|--------------------------------|---------|--------------------------------------------|
| `utils_monitor_htop_enabled`   | `true`  | Interactive process viewer                 |
| `utils_monitor_btop_enabled`   | `false` | Modern resource monitor (btop++)           |
| `utils_monitor_bottom_enabled` | `false` | System monitor (Arch only)                 |
| `utils_monitor_atop_enabled`   | `false` | Advanced process monitor with logging      |
| `utils_monitor_procs_enabled`  | `false` | Modern process viewer (Arch/Debian/EL9)    |

### Disk Usage

| Variable                  | Default | Description                              |
|---------------------------|---------|------------------------------------------|
| `utils_disk_dust_enabled` | `false` | du replacement as tree (Arch/Debian)     |
| `utils_disk_duf_enabled`  | `false` | Colourised df replacement                |

### Secrets / Encryption

| Variable                     | Default | Description                              |
|------------------------------|---------|------------------------------------------|
| `utils_secrets_age_enabled`  | `false` | Simple file encryption tool              |
| `utils_secrets_sops_enabled` | `false` | Encrypted file editor (Arch only)        |

### Network Diagnostics

| Variable                       | Default | Description                                      |
|--------------------------------|---------|--------------------------------------------------|
| `utils_net_dig_enabled`        | `false` | dig / nslookup / host (BIND tools)               |
| `utils_net_mtr_enabled`        | `false` | mtr (combined traceroute + ping)                 |
| `utils_net_traceroute_enabled` | `false` | traceroute (classic)                             |
| `utils_net_nmap_enabled`       | `false` | nmap port scanner / host discovery               |
| `utils_net_iperf3_enabled`     | `false` | iperf3 bandwidth measurement                     |
| `utils_net_netcat_enabled`     | `false` | netcat (OpenBSD flavour)                         |
| `utils_net_whois_enabled`      | `false` | whois domain registration info                   |
| `utils_net_iftop_enabled`      | `false` | iftop per-connection bandwidth TUI               |
| `utils_net_nethogs_enabled`    | `false` | nethogs per-process bandwidth                    |
| `utils_net_httpie_enabled`     | `false` | HTTPie command-line HTTP client (Arch/Debian)    |
| `utils_net_xh_enabled`         | `false` | xh rust HTTPie clone (Arch/Debian)               |
| `utils_net_arp_scan_enabled`   | `false` | arp-scan LAN host discovery (not on EL 10)       |

Package mapping per platform:

| Tool         | Arch                 | Debian Trixie    | Rocky 9      | Rocky 10     |
|--------------|----------------------|------------------|--------------|--------------|
| dig          | `bind`               | `bind9-dnsutils` | `bind-utils` | `bind-utils` |
| netcat       | `openbsd-netcat`     | `netcat-openbsd` | `netcat`     | `netcat`     |
| httpie       | `httpie`             | `httpie`         | —            | —            |
| xh           | `xh`                 | `xh`             | —            | —            |
| arp-scan     | `arp-scan`           | `arp-scan`       | `arp-scan`   | —            |

All other tools use the same package name on every supported distro.

### File Transfer

| Variable                       | Default | Description            |
|--------------------------------|---------|------------------------|
| `utils_transfer_rsync_enabled` | `true`  | Fast file copying tool |

### File Managers

| Variable                       | Default | Description                            |
|--------------------------------|---------|----------------------------------------|
| `utils_filemgr_mc_enabled`     | `true`  | Midnight Commander                     |
| `utils_filemgr_vifm_enabled`   | `false` | Vi-like file manager                   |
| `utils_filemgr_ranger_enabled` | `false` | Vi-like file manager (Arch/Debian/EL9) |
| `utils_filemgr_nnn_enabled`    | `false` | Fast file manager (Arch/Debian/EL9)    |
| `utils_filemgr_lf_enabled`     | `false` | Terminal file manager (Arch/Debian)    |

### Search Tools

| Variable                       | Default | Description                                 |
|--------------------------------|---------|---------------------------------------------|
| `utils_search_ripgrep_enabled` | `false` | Fast grep replacement (rg)                  |
| `utils_search_fd_enabled`      | `false` | Fast find replacement (fd/fdfind on Debian) |

### System Info

| Variable                        | Default | Description                                 |
|---------------------------------|---------|---------------------------------------------|
| `utils_fetch_fastfetch_enabled` | `false` | Fast system info tool (C)                   |
| `utils_fetch_hyfetch_enabled`   | `false` | Maintained neofetch successor (Arch/Debian) |
| `utils_fetch_archey4_enabled`   | `false` | System info script (Arch AUR only)          |
| `utils_fetch_cpufetch_enabled`  | `false` | CPU info tool (Arch only)                   |

### Fun Tools

| Variable                                 | Default | Description                            |
|------------------------------------------|---------|----------------------------------------|
| `utils_fun_cowsay_enabled`               | `false` | Talking cow                            |
| `utils_fun_fortune_enabled`              | `false` | Random fortune cookies                 |
| `utils_fun_lolcat_enabled`               | `false` | Rainbow text (Arch/Debian)             |
| `utils_fun_ponysay_enabled`              | `false` | Pony cowsay (Arch/Debian)              |
| `utils_fun_figlet_enabled`               | `false` | Large text banners                     |
| `utils_fun_toilet_enabled`               | `false` | Colored text banners                   |
| `utils_fun_sl_enabled`                   | `false` | Steam locomotive                       |
| `utils_fun_cmatrix_enabled`              | `false` | Matrix screensaver (Arch/Debian/EL9)   |
| `utils_fun_cbonsai_enabled`              | `false` | Bonsai tree (Arch AUR/Debian/EL9)      |
| `utils_fun_asciiquarium_enabled`         | `false` | ASCII aquarium (Arch/Debian/EL9)       |
| `utils_fun_nyancat_enabled`              | `false` | Nyan Cat (Arch/Debian/EL9)             |
| `utils_fun_no_more_secrets_enabled`      | `false` | Sneakers effect (Arch AUR/Debian/EL9)  |
| `utils_fun_tty_clock_enabled`            | `false` | Terminal clock (Arch AUR/Debian/EL9)   |
| `utils_fun_pipes_enabled`                | `false` | Animated pipes (Arch AUR only)         |
| `utils_fun_pokemon_colorscripts_enabled` | `false` | Pokemon scripts (Arch AUR only)        |
| `utils_fun_pokemonsay_enabled`           | `false` | Pokemon cowsay (Arch AUR only)         |
| `utils_fun_hollywood_enabled`            | `false` | Fake hacker terminal (Arch AUR/Debian) |

## Tags

| Tag             | Scope                |
|-----------------|----------------------|
| `utils`         | All role tasks       |
| `utils:install` | Package installation |

## Example Playbook

```yaml
- name: Include utils role
  ansible.builtin.include_role:
    name: marcstraube.common.utils
  tags: [utils]
  when: utils_enabled | default(true) | bool
```

## Testing

```bash
cd roles/utils
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## References

- [btop](https://github.com/aristocratos/btop) — modern resource monitor for the terminal
- [ripgrep](https://github.com/BurntSushi/ripgrep) — fast recursive grep alternative
- [fd](https://github.com/sharkdp/fd) — fast find alternative
- [procs](https://github.com/dalance/procs) — modern `ps` replacement with colourised output
- [dust](https://github.com/bootandy/dust) — `du` replacement showing disk usage as a tree
- [duf](https://github.com/muesli/duf) — `df` replacement with colourised filesystem overview
- [age](https://github.com/FiloSottile/age) — simple file encryption tool
- [sops](https://github.com/getsops/sops) — editor for encrypted YAML/JSON/ENV/INI files
- [BIND tools](https://www.isc.org/bind/) — dig, nslookup, host
- [mtr](https://github.com/traviscross/mtr) — combined traceroute and ping
- [nmap](https://nmap.org/) — Network Mapper, port scanner
- [iperf3](https://iperf.fr/) — bandwidth measurement
- [OpenBSD netcat](https://man.openbsd.org/nc) — TCP/UDP swiss army knife
- [whois](https://www.iana.org/whois) — domain registration lookup
- [iftop](https://www.ex-parrot.com/pdw/iftop/) — per-connection bandwidth TUI
- [nethogs](https://github.com/raboof/nethogs) — per-process bandwidth
- [HTTPie](https://httpie.io/) — modern command-line HTTP client
- [xh](https://github.com/ducaale/xh) — rust HTTPie reimplementation
- [arp-scan](https://github.com/royhills/arp-scan) — LAN host discovery via ARP

## License

MIT

## Author

Marc Straube
