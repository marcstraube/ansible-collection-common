# marcstraube.common.fonts

Install and configure system fonts.

## Description

Installs base fonts (DejaVu, Liberation, Noto), coding fonts (Fira Code, JetBrains
Mono, etc.), and Nerd Fonts. Manages fontconfig rendering settings including
antialiasing, hinting, subpixel rendering, and font family preferences. Supports
Arch Linux, Debian Trixie, and Rocky Linux 9/10.

## Requirements

- ansible-core >= 2.17
- No additional collections required

## Supported Platforms

| Platform       | Notes                                   |
| -------------- | --------------------------------------- |
| Arch Linux     | Nerd Fonts via community packages       |
| Debian Trixie  | Nerd Fonts via packages where available |
| Rocky Linux 9  | Nerd Fonts via packages where available |
| Rocky Linux 10 | Nerd Fonts via packages where available |

## Role Variables

### Role Control

| Variable        | Default | Description             |
| --------------- | ------- | ----------------------- |
| `fonts_enabled` | `true`  | Enable/disable the role |

### Base Fonts

| Variable           | Default | Description                                   |
| ------------------ | ------- | --------------------------------------------- |
| `fonts_base`       | `true`  | Install base font set                         |
| `fonts_dejavu`     | `true`  | Install DejaVu fonts                          |
| `fonts_liberation` | `true`  | Install Liberation fonts (MS font compatible) |
| `fonts_noto`       | `true`  | Install Noto fonts (Google Unicode)           |
| `fonts_noto_emoji` | `true`  | Install Noto Emoji fonts                      |
| `fonts_noto_cjk`   | `false` | Install Noto CJK fonts (~1.5 GB)              |

### Coding Fonts

| Variable                | Default                     | Description                     |
| ----------------------- | --------------------------- | ------------------------------- |
| `fonts_coding`          | `false`                     | Install coding/monospace fonts  |
| `fonts_coding_families` | `['firacode', 'jetbrains']` | Coding font families to install |

Available families: `firacode`, `jetbrains`, `cascadia`, `iosevka`, `hack`, `source-code-pro`.

### Nerd Fonts

| Variable              | Default            | Description                             |
| --------------------- | ------------------ | --------------------------------------- |
| `fonts_nerd`          | `false`            | Install Nerd Fonts (patched with icons) |
| `fonts_nerd_families` | `['symbols-only']` | Nerd Font families to install           |

Available families: `firacode`, `hack`, `jetbrains`, `meslo`, `source-code-pro`,
`ubuntu`, `dejavu`, `liberation`, `noto`, `roboto`, `droid`, `terminus`, `symbols-only`.

### Additional Fonts

| Variable               | Default | Description                          |
| ---------------------- | ------- | ------------------------------------ |
| `fonts_extra_packages` | `[]`    | Additional OS-specific font packages |

### Fontconfig

| Variable                        | Default      | Description                                                 |
| ------------------------------- | ------------ | ----------------------------------------------------------- |
| `fonts_fontconfig_enabled`      | `true`       | Enable fontconfig customization                             |
| `fonts_fontconfig_prefer_sans`  | `''`         | Preferred sans-serif font                                   |
| `fonts_fontconfig_prefer_serif` | `''`         | Preferred serif font                                        |
| `fonts_fontconfig_prefer_mono`  | `''`         | Preferred monospace font                                    |
| `fonts_fontconfig_antialias`    | `true`       | Enable antialiasing                                         |
| `fonts_fontconfig_hinting`      | `hintslight` | Hinting: `hintnone`, `hintslight`, `hintmedium`, `hintfull` |
| `fonts_fontconfig_subpixel`     | `rgb`        | Subpixel rendering: `none`, `rgb`, `bgr`, `vrgb`, `vbgr`    |
| `fonts_fontconfig_lcdfilter`    | `default`    | LCD filter: `none`, `default`, `light`, `legacy`            |

### Cache

| Variable              | Default | Description                           |
| --------------------- | ------- | ------------------------------------- |
| `fonts_rebuild_cache` | `true`  | Rebuild font cache after installation |

## Tags

| Tag             | Scope                         |
| --------------- | ----------------------------- |
| `fonts`         | All role tasks                |
| `fonts:install` | Font package installation     |
| `fonts:config`  | Fontconfig rendering settings |

## Example Playbook

```yaml
- name: Configure fonts
  hosts: workstations
  become: true
  tasks:
    - name: Include fonts role
      ansible.builtin.include_role:
        name: marcstraube.common.fonts
      tags: [fonts]
      when: fonts_enabled | default(true) | bool
```

### Desktop with Coding and Nerd Fonts

```yaml
- name: Include fonts role
  ansible.builtin.include_role:
    name: marcstraube.common.fonts
  vars:
    fonts_coding: true
    fonts_coding_families:
      - 'firacode'
      - 'jetbrains'
      - 'iosevka'
    fonts_nerd: true
    fonts_nerd_families:
      - 'symbols-only'
      - 'firacode'
    fonts_fontconfig_prefer_mono: 'FiraCode Nerd Font'
```

## Testing

```bash
cd roles/fonts
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## License

MIT

## Author

Marc Straube
