# marcstraube.common.graphics

Install GPU drivers, Vulkan, video acceleration, and hybrid graphics (PRIME).

## Description

Manages GPU driver installation for Intel, AMD, and NVIDIA graphics cards across
Arch Linux, Debian, and RHEL/Rocky. Supports auto-detection of GPU hardware,
hybrid graphics (PRIME render offload), and NVIDIA RTD3 power management.

## Requirements

- ansible-core >= 2.20
- `lspci` available on target (part of `pciutils`) when using `graphics_gpu: auto`
- RPM Fusion repositories for NVIDIA on RHEL/Rocky
- AUR helper for legacy NVIDIA drivers on Arch Linux

## Supported Platforms

- Arch Linux (rolling)
- Debian Trixie (13)
- Rocky Linux 9 / 10

## Role Variables

### Role Control

| Variable           | Default | Description                                  |
| ------------------ | ------- | -------------------------------------------- |
| `graphics_enabled` | `true`  | Enable/disable the entire role               |
| `graphics_gpu`     | `auto`  | GPU vendor: `intel`, `amd`, `nvidia`, `auto` |
| `graphics_gpus`    | `[]`    | Multiple GPUs (e.g. `['intel', 'nvidia']`)   |

### Intel GPU

| Variable                      | Default              | Description                       |
| ----------------------------- | -------------------- | --------------------------------- |
| `graphics_intel_enabled`      | `false`              | Install Intel GPU drivers         |
| `graphics_intel_mesa`         | `true`               | Mesa 3D driver                    |
| `graphics_intel_vulkan`       | `true`               | Vulkan support (Broadwell+)       |
| `graphics_intel_vaapi`        | `true`               | VA-API hardware video accel       |
| `graphics_intel_vaapi_driver` | `intel-media-driver` | VA-API driver choice              |
| `graphics_intel_guc_huc`      | `false`              | GuC/HuC firmware (Gen 9+ Skylake) |
| `graphics_intel_32bit`        | `false`              | 32-bit support (Steam, Wine)      |

### AMD GPU

| Variable               | Default | Description                  |
| ---------------------- | ------- | ---------------------------- |
| `graphics_amd_enabled` | `false` | Install AMD GPU drivers      |
| `graphics_amd_mesa`    | `true`  | Mesa 3D driver               |
| `graphics_amd_vulkan`  | `true`  | Vulkan support               |
| `graphics_amd_32bit`   | `false` | 32-bit support (Steam, Wine) |

### NVIDIA GPU

| Variable                         | Default | Description                              |
| -------------------------------- | ------- | ---------------------------------------- |
| `graphics_nvidia_enabled`        | `false` | Install NVIDIA GPU drivers               |
| `graphics_nvidia_driver`         | `open`  | Driver: `open`, `proprietary`, `nouveau` |
| `graphics_nvidia_legacy_version` | `''`    | Legacy driver: `580xx`, `470xx`, `390xx` |
| `graphics_nvidia_32bit`          | `false` | 32-bit support                           |
| `graphics_nvidia_cuda`           | `false` | CUDA toolkit                             |

### Hybrid Graphics (PRIME)

| Variable                              | Default | Description                                   |
| ------------------------------------- | ------- | --------------------------------------------- |
| `graphics_hybrid_enabled`             | `false` | Enable hybrid graphics support                |
| `graphics_hybrid_switcheroo`          | `true`  | Install switcheroo-control for DE integration |
| `graphics_hybrid_prime_run`           | `true`  | Install prime-run helper (Arch)               |
| `graphics_hybrid_nvidia_rtd3`         | `false` | NVIDIA RTD3 power management                  |
| `graphics_hybrid_nvidia_rtd3_level`   | `0x02`  | RTD3 level: `0x02` (Turing), `0x03` (Ampere+) |
| `graphics_hybrid_nvidia_persistenced` | `false` | Enable nvidia-persistenced service            |

### Common Tools

| Variable                | Default | Description              |
| ----------------------- | ------- | ------------------------ |
| `graphics_vulkan_tools` | `true`  | Vulkan loader and tools  |
| `graphics_vaapi_tools`  | `true`  | VA-API tools (vainfo)    |
| `graphics_vdpau_tools`  | `false` | VDPAU tools (vdpauinfo)  |
| `graphics_mesa_utils`   | `true`  | Mesa utilities (glxinfo) |

### Kernel Parameters

| Variable                         | Default | Description               |
| -------------------------------- | ------- | ------------------------- |
| `graphics_kernel_params_enabled` | `false` | Deploy modprobe.d configs |
| `graphics_intel_kernel_params`   | `[]`    | i915 module options       |
| `graphics_nvidia_kernel_params`  | `[]`    | nvidia module options     |

## Tags

| Tag                  | Scope                          |
| -------------------- | ------------------------------ |
| `graphics`           | All tasks                      |
| `graphics:detect`    | GPU auto-detection             |
| `graphics:intel`     | Intel driver installation      |
| `graphics:amd`       | AMD driver installation        |
| `graphics:nvidia`    | NVIDIA driver installation     |
| `graphics:tools`     | Common tools installation      |
| `graphics:hybrid`    | Hybrid graphics / PRIME setup  |
| `graphics:configure` | Kernel parameter configuration |

## Example Playbook

```yaml
- hosts: workstations
  roles:
    - role: marcstraube.common.graphics
      vars:
        graphics_intel_enabled: true
        graphics_nvidia_enabled: true
        graphics_hybrid_enabled: true
        graphics_hybrid_nvidia_rtd3: true
        graphics_hybrid_nvidia_rtd3_level: '0x02'
        graphics_hybrid_nvidia_persistenced: true
```

## Testing

```bash
cd roles/graphics
molecule test
```

Driver: `podman` | Platforms: Arch Linux, Debian Trixie, Rocky 9, Rocky 10

## NVIDIA Driver Notes

- **Arch Linux**: Since Dec 2025, `nvidia-open` is the only official driver.
  Legacy GPUs need AUR packages (`nvidia-580xx-dkms`, `nvidia-470xx-dkms`).
- **Debian Trixie**: Legacy 470xx driver has been dropped.
- **RHEL/Rocky**: Requires RPM Fusion repositories for proprietary NVIDIA drivers.

## License

MIT

## Author

Marc Straube
