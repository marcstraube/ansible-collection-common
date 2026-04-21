# marcstraube.common.fonts

Install and configure system fonts.

## Description

Installs base fonts (DejaVu, Liberation, Noto), coding fonts (Fira Code, JetBrains
Mono, etc.), and Nerd Fonts. Manages fontconfig rendering settings including
antialiasing, hinting, subpixel rendering, and font family preferences.

## Requirements

- ansible-core >= 2.17
- No additional collections required

## Supported Platforms

| Platform                  | Notes                             |
|---------------------------|-----------------------------------|
| Arch Linux                | Nerd Fonts via community packages |
| Debian Trixie             |                                   |
| EL 9 (Rocky, Alma, RHEL)  |                                   |
| EL 10 (Rocky, Alma, RHEL) |                                   |

Other distributions in the same os_family (EndeavourOS, Manjaro, Ubuntu, Mint,
Fedora) should work but are not actively tested. Use distro-specific vars
overrides if needed.

## Role Variables

### Role Control

| Variable        | Default | Description           |
|-----------------|---------|-----------------------|
| `fonts_enabled` | `true`  | Enable the fonts role |

### Base Fonts

| Variable                   | Default | Description                                  |
|----------------------------|---------|----------------------------------------------|
| `fonts_base_enabled`       | `true`  | Enable base font set                         |
| `fonts_dejavu_enabled`     | `true`  | Enable DejaVu fonts                          |
| `fonts_liberation_enabled` | `true`  | Enable Liberation fonts (MS font compatible) |
| `fonts_noto_enabled`       | `true`  | Enable Noto fonts (Google Unicode)           |
| `fonts_noto_emoji_enabled` | `true`  | Enable Noto Emoji fonts                      |
| `fonts_noto_cjk_enabled`   | `false` | Enable Noto CJK fonts (~1.5 GB)              |

### Coding Fonts

| Variable                | Default                     | Description                     |
|-------------------------|-----------------------------|---------------------------------|
| `fonts_coding_enabled`  | `false`                     | Enable coding/monospace fonts   |
| `fonts_coding_families` | `['firacode', 'jetbrains']` | Coding font families to install |

Available families: `firacode`, `jetbrains`, `cascadia`, `iosevka`, `hack`, `source-code-pro`.

### Nerd Fonts

| Variable              | Default            | Description                            |
|-----------------------|--------------------|----------------------------------------|
| `fonts_nerd_enabled`  | `false`            | Enable Nerd Fonts (patched with icons) |
| `fonts_nerd_families` | `['symbols-only']` | Nerd Font families to install          |

Available families: `firacode`, `hack`, `jetbrains`, `meslo`, `source-code-pro`,
`ubuntu`, `dejavu`, `liberation`, `noto`, `roboto`, `droid`, `terminus`, `symbols-only`.

### Additional Fonts

| Variable               | Default | Description                          |
|------------------------|---------|--------------------------------------|
| `fonts_extra_packages` | `[]`    | Additional OS-specific font packages |

### Fontconfig

| Variable                             | Default      | Description                     |
|--------------------------------------|--------------|---------------------------------|
| `fonts_fontconfig_enabled`           | `true`       | Enable fontconfig customization |
| `fonts_fontconfig_prefer_sans`       | `''`         | Preferred sans-serif font       |
| `fonts_fontconfig_prefer_serif`      | `''`         | Preferred serif font            |
| `fonts_fontconfig_prefer_mono`       | `''`         | Preferred monospace font        |
| `fonts_fontconfig_antialias_enabled` | `true`       | Enable antialiasing             |
| `fonts_fontconfig_hinting`           | `hintslight` | Hinting style                   |
| `fonts_fontconfig_subpixel`          | `rgb`        | Subpixel rendering              |
| `fonts_fontconfig_lcdfilter`         | `default`    | LCD filter                      |

Hinting: `hintnone`, `hintslight`, `hintmedium`, `hintfull`.
Subpixel: `none`, `rgb`, `bgr`, `vrgb`, `vbgr`.
LCD filter: `none`, `default`, `light`, `legacy`.

### Cache

| Variable                      | Default | Description                             |
|-------------------------------|---------|-----------------------------------------|
| `fonts_rebuild_cache_enabled` | `true`  | Enable font cache rebuild after install |

## Tags

| Tag             | Scope                         |
|-----------------|-------------------------------|
| `fonts`         | All role tasks                |
| `fonts:install` | Font package installation     |
| `fonts:config`  | Fontconfig rendering settings |

## Example Playbook

```yaml
- name: Include fonts role
  ansible.builtin.include_role:
    name: marcstraube.common.fonts
  tags:
    - fonts
  when: fonts_enabled | default(true) | bool
```

### Desktop with Coding and Nerd Fonts

```yaml
- name: Include fonts role
  ansible.builtin.include_role:
    name: marcstraube.common.fonts
  vars:
    fonts_coding_enabled: true
    fonts_coding_families:
      - 'firacode'
      - 'jetbrains'
      - 'iosevka'
    fonts_nerd_enabled: true
    fonts_nerd_families:
      - 'symbols-only'
      - 'firacode'
    fonts_fontconfig_prefer_mono: 'FiraCode Nerd Font'
  tags:
    - fonts
  when: fonts_enabled | default(true) | bool
```

## Testing

```bash
cd roles/fonts
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## References

- [fontconfig](https://www.freedesktop.org/wiki/Software/fontconfig/) — Font configuration and customization library
- [Nerd Fonts](https://www.nerdfonts.com/) — Developer-targeted patched fonts with icon glyphs

## License

MIT

## Author

Marc Straube
