# marcstraube.common.graphics

Install GPU drivers, Vulkan, video acceleration, and hybrid graphics (PRIME).

## Description

Manages GPU driver installation for Intel, AMD, and NVIDIA graphics cards across
Arch Linux, Debian, and RHEL/Rocky. Supports auto-detection of GPU hardware,
hybrid graphics (PRIME render offload), and NVIDIA RTD3 power management.

## Requirements

- ansible-core >= 2.17
- `lspci` available on target (part of `pciutils`) when using `graphics_gpu: auto`
- RPM Fusion repositories for NVIDIA on RHEL/Rocky
- AUR helper for legacy NVIDIA drivers on Arch Linux

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

| Variable           | Default | Description              |
|--------------------|---------|--------------------------|
| `graphics_enabled` | `true`  | Enable the graphics role |

### GPU Vendor Selection

| Variable        | Default | Description                                  |
|-----------------|---------|----------------------------------------------|
| `graphics_gpu`  | `auto`  | GPU vendor: `intel`, `amd`, `nvidia`, `auto` |
| `graphics_gpus` | `[]`    | Multiple GPUs (e.g. `['intel', 'nvidia']`)   |

### Intel GPU

| Variable                         | Default              | Description                      |
|----------------------------------|----------------------|----------------------------------|
| `graphics_intel_enabled`         | `false`              | Enable Intel GPU drivers         |
| `graphics_intel_mesa_enabled`    | `true`               | Enable Mesa 3D driver            |
| `graphics_intel_vulkan_enabled`  | `true`               | Enable Vulkan (Broadwell+)       |
| `graphics_intel_vaapi_enabled`   | `true`               | Enable VA-API video acceleration |
| `graphics_intel_vaapi_driver`    | `intel-media-driver` | VA-API driver choice             |
| `graphics_intel_guc_huc_enabled` | `false`              | Enable GuC/HuC firmware (Gen 9+) |
| `graphics_intel_32bit_enabled`   | `false`              | Enable 32-bit support            |

### AMD GPU

| Variable                      | Default | Description            |
|-------------------------------|---------|------------------------|
| `graphics_amd_enabled`        | `false` | Enable AMD GPU drivers |
| `graphics_amd_mesa_enabled`   | `true`  | Enable Mesa 3D driver  |
| `graphics_amd_vulkan_enabled` | `true`  | Enable Vulkan support  |
| `graphics_amd_32bit_enabled`  | `false` | Enable 32-bit support  |

### NVIDIA GPU

| Variable                         | Default | Description                              |
|----------------------------------|---------|------------------------------------------|
| `graphics_nvidia_enabled`        | `false` | Enable NVIDIA GPU drivers                |
| `graphics_nvidia_driver`         | `open`  | Driver: `open`, `proprietary`, `nouveau` |
| `graphics_nvidia_legacy_version` | `''`    | Legacy driver: `580xx`, `470xx`, `390xx` |
| `graphics_nvidia_32bit_enabled`  | `false` | Enable 32-bit support                    |
| `graphics_nvidia_cuda_enabled`   | `false` | Enable CUDA toolkit                      |

### Common Tools

| Variable                        | Default | Description                    |
|---------------------------------|---------|--------------------------------|
| `graphics_vulkan_tools_enabled` | `true`  | Enable Vulkan loader and tools |
| `graphics_vaapi_tools_enabled`  | `true`  | Enable VA-API tools (vainfo)   |
| `graphics_vdpau_tools_enabled`  | `false` | Enable VDPAU tools (vdpauinfo) |
| `graphics_mesa_utils_enabled`   | `true`  | Enable Mesa utilities          |

### Hybrid Graphics (PRIME)

| Variable                                      | Default | Description                         |
|-----------------------------------------------|---------|-------------------------------------|
| `graphics_hybrid_enabled`                     | `false` | Enable hybrid graphics support      |
| `graphics_hybrid_switcheroo_enabled`          | `true`  | Enable switcheroo-control           |
| `graphics_hybrid_prime_run_enabled`           | `true`  | Enable prime-run helper (Arch)      |
| `graphics_hybrid_nvidia_rtd3_enabled`         | `false` | Enable NVIDIA RTD3 power management |
| `graphics_hybrid_nvidia_rtd3_level`           | `0x02`  | RTD3 level: `0x02`/`0x03`           |
| `graphics_hybrid_nvidia_persistenced_enabled` | `false` | Enable nvidia-persistenced          |

### Kernel Parameters

| Variable                         | Default | Description               |
|----------------------------------|---------|---------------------------|
| `graphics_kernel_params_enabled` | `false` | Enable modprobe.d configs |
| `graphics_intel_kernel_params`   | `[]`    | i915 module options       |
| `graphics_nvidia_kernel_params`  | `[]`    | nvidia module options     |

## Tags

| Tag                  | Scope                          |
|----------------------|--------------------------------|
| `graphics`           | All tasks                      |
| `graphics:detect`    | GPU auto-detection             |
| `graphics:intel`     | Intel driver install           |
| `graphics:amd`       | AMD driver install             |
| `graphics:nvidia`    | NVIDIA driver install          |
| `graphics:tools`     | Common tools install           |
| `graphics:hybrid`    | Hybrid graphics / PRIME setup  |
| `graphics:configure` | Kernel parameter configuration |

## Example Playbook

```yaml
- name: Include graphics role
  ansible.builtin.include_role:
    name: marcstraube.common.graphics
  tags:
    - graphics
  when: graphics_enabled | default(true) | bool
```

## Testing

```bash
cd roles/graphics
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## Notes

- **Arch Linux**: Since Dec 2025, `nvidia-open` is the only official driver.
  Legacy GPUs need AUR packages (`nvidia-580xx-dkms`, `nvidia-470xx-dkms`).
- **Debian Trixie**: Legacy 470xx driver has been dropped.
- **RHEL/Rocky**: Requires RPM Fusion repositories for proprietary NVIDIA drivers.

## License

MIT

## Author

Marc Straube
