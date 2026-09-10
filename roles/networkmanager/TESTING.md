# Testing NetworkManager Role

This role uses [Molecule](https://molecule.readthedocs.io/) for comprehensive testing of all NetworkManager features.

## Prerequisites

### Required Software

```bash
# Install Molecule and Ansible
pip install molecule molecule-plugins ansible-core

# Or on Arch Linux
yay -S molecule molecule-plugins ansible-core

# Ensure Vagrant and VirtualBox are installed
vagrant --version
VBoxManage --version
```

### System Requirements

- **Vagrant**: >= 2.3.0
- **VirtualBox**: >= 6.1
- **Python**: >= 3.11
- **Disk Space**: ~2GB for VM and images

## Test Platform

- **Driver**: Vagrant with VirtualBox provider

| Platform      | Box                             | Resources       | Test Scope                           |
|---------------|---------------------------------|-----------------|--------------------------------------|
| Arch Linux    | `marcstraube/archlinux-ansible` | 1024 MB, 2 CPUs | Full set incl. WiFi, VPN, SMB, SSHFS |
| Debian Trixie | `cloud-image/debian-13`         | 768 MB, 1 CPU   | Ethernet only                        |
| Rocky 9       | `bento/rockylinux-9`            | 768 MB, 1 CPU   | Ethernet only                        |
| Rocky 10      | `bento/rockylinux-10`           | 768 MB, 1 CPU   | Ethernet only                        |

**Network setup:**

- eth0: NAT (default Vagrant interface, all platforms)
- eth1: Private network, DHCP (all platforms)
- eth2: Private network, DHCP (Arch Linux only, for secondary connections)
- wlan0-wlan3: Virtual WiFi radios using mac80211_hwsim kernel module (Arch Linux only)

## Test Coverage

The test suite validates all features of this role:

### 1. Basic NetworkManager Installation

- Package installation
- Service running and enabled
- nmcli command availability

### 2. Connection Management

Tests creation and configuration of:

- **Ethernet connections** with static IP
- **WiFi connections** with SSID configuration
- **VPN connections** (PPTP for testing)

### 3. VPN Auto-Connect (connection.secondaries)

Verifies that VPN connections are automatically linked to primary
connections using NetworkManager's `connection.secondaries` feature:

- Ethernet → VPN auto-connect
- WiFi → VPN auto-connect
- UUID resolution and configuration

### 4. DHCP Hostname Sending

Validates granular control over DHCP hostname sending:

- Global configuration for IPv4 and IPv6
- Per-connection overrides
- Correct nmcli property settings

### 5. SMB/CIFS Share Auto-Mounting

Tests automatic mounting of SMB shares with:

- Connection-dependent mounting (Ethernet, WiFi, VPN)
- VPN-dependent vs non-VPN shares separation
- Correct dispatcher script generation
- Mount/umount on connection up/down events

### 6. SSHFS Share Auto-Mounting

Similar to SMB, validates SSHFS mounting with:

- SSH-based filesystem mounts
- Connection dependencies
- Dispatcher script generation

### 7. Unmanaged Devices

Verifies configuration of devices that NetworkManager should ignore:

- MAC address-based exclusion
- Interface name-based exclusion
- Wildcard patterns (e.g., `veth*`)

### 8. Dispatcher Scripts

Validates all dispatcher scripts:

- Correct file permissions (0755)
- Bash syntax validation
- Proper event handling (up, down, vpn-up, vpn-down)
- Pre-down script placement

### 9. Idempotence

Ensures the role is fully idempotent by running converge twice and verifying no changes on the second run.

## Running Tests

### Full Test Suite

```bash
# Run complete test sequence (recommended)
molecule test
```

This executes:

1. **Dependency installation** - Install required collections
2. **Syntax check** - Validate YAML syntax
3. **VM creation** - Create VirtualBox VM with Vagrant
4. **Preparation** - Update system, setup WiFi radios, start access points
5. **Converge** - Apply the role
6. **Idempotence** - Verify no changes on second run
7. **Verification** - Run comprehensive tests (verify.yml)
8. **Cleanup & Destroy** - Remove test VM

### Individual Test Steps

```bash
# Create and prepare test VM
molecule create
molecule prepare

# Apply the role
molecule converge

# Run verification tests
molecule verify

# Test idempotence (should show 0 changes)
molecule idempotence

# Login to test VM for manual inspection
molecule login

# Destroy test VM
molecule destroy
```

### Manual Inspection Commands

After `molecule login`, you can run commands interactively in the VM:

```bash
# View all connections
nmcli connection show

# Show specific connection details
nmcli connection show 'Test Ethernet Connection'

# Check VPN auto-connect configuration (connection.secondaries)
nmcli -t -f connection.secondaries connection show 'Test Ethernet Connection'

# List dispatcher scripts
ls -la /etc/NetworkManager/dispatcher.d/

# View dispatcher script content
cat /etc/NetworkManager/dispatcher.d/30-mount-smb.sh

# View Ansible-managed daemon configuration
cat /etc/NetworkManager/conf.d/00-ansible.conf

# List available WiFi networks
nmcli device wifi list

# Show NetworkManager status
systemctl status NetworkManager
```

### Quick Development Cycle

For rapid iteration during development:

```bash
# One-time setup
molecule create
molecule prepare  # Takes ~5 minutes (system update, WiFi setup)

# Rapid development loop
molecule converge  # Apply changes (~30 seconds)
molecule verify    # Run tests (~20 seconds)

# Manual inspection
molecule login

# When done
molecule destroy
```

## Test Configuration

All test variables are centralized in `molecule/default/molecule.yml` under `provisioner.inventory.group_vars.all`:

```yaml
# Example test configuration
test_networkmanager_connections:
  ethernet:
    'Test Ethernet Connection':
      ifname: eth1
      ip4: 192.168.1.10/24
      vpn:
        - Test VPN Company
      settings:
        ipv4.dhcp-send-hostname: "yes"
  wifi:
    'Office WiFi':
      ifname: wlan2
      ssid: Office-Network
      vpn:
        - Test VPN Company
  vpn:
    'Test VPN Company':
      vpn:
        service-type: org.freedesktop.NetworkManager.pptp
        gateway: vpn.example.com
```

## Virtual WiFi Setup

The test environment creates a realistic WiFi environment using the Linux `mac80211_hwsim` kernel module:

- **4 virtual WiFi radios** are created (by Vagrantfile provisioning)
- **2 access points** (wlan0, wlan1) broadcast test SSIDs during prepare phase:
  - `Office-Network` (channel 6)
  - `Home-Network` (channel 11)
- **2 client interfaces** (wlan2, wlan3) connect to these APs
- **hostapd** simulates real access points

This setup allows testing WiFi connections without physical hardware.

## Troubleshooting

### VM Stuck in Poweroff State

```bash
molecule destroy
molecule create
```

### Vagrant Box Not Found

```bash
vagrant box add marcstraube/archlinux-ansible
```

### GPG Key Errors (Arch Linux)

The prepare phase handles this automatically by:

1. Initializing pacman keyring
2. Updating `archlinux-keyring` first
3. Then performing full system update

### WiFi Interfaces Not Created

Login to VM and check if mac80211_hwsim module loaded:

```bash
molecule login
# Then in the VM:
lsmod | grep mac80211_hwsim
iw dev
```

Or use Vagrant directly. Molecule creates the VM in an ephemeral directory below
`~/.ansible/tmp/`, named `molecule.<checksum>.<scenario>` after the role directory:

```bash
ls -d ~/.ansible/tmp/molecule.*.default   # pick the one holding this role's Vagrantfile
cd <ephemeral directory>
vagrant ssh
```

### Connection Test Failures

Inspect the actual connection configuration:

```bash
molecule login
# Then in the VM:
nmcli connection show 'Test Ethernet Connection'
nmcli -t -f connection.secondaries connection show 'Test Ethernet Connection'
```

### Dispatcher Script Issues

Check script syntax and content:

```bash
molecule login
# Then in the VM:
bash -n /etc/NetworkManager/dispatcher.d/30-mount-smb.sh
cat /etc/NetworkManager/dispatcher.d/30-mount-smb.sh
```

### View Molecule Cache and VM Files

```bash
ls -d ~/.ansible/tmp/molecule.*.default   # pick the one holding this role's Vagrantfile
cd <ephemeral directory>
ls -la
vagrant status
```

### View Test Results in Detail

```bash
molecule --debug converge
molecule --debug verify
```

### Hostapd Not Starting

Check hostapd logs in the VM:

```bash
molecule login
# Then in the VM:
cat /tmp/hostapd_office.log
cat /tmp/hostapd_home.log
pgrep -a hostapd
```

## Customizing Tests

### Testing on Different Distributions

Edit `molecule/default/Vagrantfile`:

```ruby
# For Debian
config.vm.box = "generic/debian12"

# For Ubuntu
config.vm.box = "generic/ubuntu2404"

# For Fedora
config.vm.box = "generic/fedora39"
```

Then update `molecule/default/prepare.yml` to use appropriate package manager (apt, dnf, etc.).

### Adjusting VM Resources

Edit `molecule/default/Vagrantfile`:

```ruby
vb.memory = "2048"  # 2GB RAM
vb.cpus = 4         # 4 CPU cores
```

### Adding Test Cases

1. Add test variables to `molecule/default/molecule.yml`
2. Update `molecule/default/converge.yml` if needed
3. Add verification tasks to `molecule/default/verify.yml`

Example - adding a new connection test:

```yaml
# In molecule.yml
test_networkmanager_connections:
  ethernet:
    'New Test Connection':
      ifname: eth2
      ip4: 192.168.3.10/24
      gw4: 192.168.3.1
```

Then add verification in `verify.yml`:

```yaml
- name: Verify new connection exists
  ansible.builtin.assert:
    that:
      - "'New Test Connection' in nmcli_connections.stdout"
    fail_msg: "Connection 'New Test Connection' was not created"
```

## CI/CD Integration

This role is not covered by the collection's GitHub Actions workflow. Molecule runs
there use the Podman driver, while this role needs a full virtual machine with
VirtualBox for its virtual WiFi radios. `.github/workflows/ci.yml` therefore excludes
the Vagrant-based roles (`apparmor`, `auditd`, `hardening`, `hardware_tokens`,
`networkmanager`, `wireguard`) from the Molecule matrix — run their tests locally
before committing.

## Test Maintenance

### Keeping Tests Up-to-Date

1. **Update Vagrant box regularly**:

   ```bash
   vagrant box update marcstraube/archlinux-ansible
   ```

2. **Review test variables** when adding new features to the role

3. **Update verify.yml** to test new functionality

### Performance Optimization

- **Preparation phase** takes ~5 minutes (system update + WiFi AP setup)
- **Converge phase** takes ~30 seconds
- **Verify phase** takes ~20 seconds
- **Full test** takes ~7-10 minutes

To speed up development:

- Keep VM running between test iterations
- Use `molecule converge && molecule verify` instead of `molecule test`
- Only run `molecule test` before committing

## Architecture Details

### Why marcstraube/archlinux-ansible Box?

This custom Vagrant box offers several advantages over `generic/arch`:

- **More up-to-date**: Regular updates with latest packages
- **Pre-installed Python3**: Saves setup time, no need to install in prepare phase
- **Ansible-ready**: Optimized for Ansible testing
- **Faster convergence**: Less preparation overhead

### Network Interface Setup

The Vagrantfile creates a realistic network environment:

```ruby
# eth0: Default Vagrant NAT (always present)
# eth1: Private network (for testing primary connections)
node.vm.network "private_network", type: "dhcp"
# eth2: Private network (for testing secondary connections)
node.vm.network "private_network", type: "dhcp"
```

WiFi interfaces (wlan0-wlan3) are created by the mac80211_hwsim kernel module,
which is loaded during Vagrant provisioning.

### Test Workflow

```text
┌─────────────┐
│ molecule    │
│   create    │──→ Vagrantfile provisions VM
└─────────────┘    ├─ Loads mac80211_hwsim
                   ├─ Creates 4 WiFi radios
                   └─ Installs iw if needed
        │
        ▼
┌─────────────┐
│ molecule    │
│  prepare    │──→ prepare.yml runs
└─────────────┘    ├─ Updates archlinux-keyring
                   ├─ Full system update
                   ├─ Installs hostapd
                   ├─ Starts WiFi APs (Office-Network, Home-Network)
                   └─ Scans for WiFi networks
        │
        ▼
┌─────────────┐
│ molecule    │
│  converge   │──→ converge.yml runs
└─────────────┘    ├─ Ignores eth0 (Vagrant interface)
                   └─ Applies marcstraube.common.networkmanager role
        │
        ▼
┌─────────────┐
│ molecule    │
│   verify    │──→ verify.yml validates everything
└─────────────┘    ├─ Service status
                   ├─ Connections created
                   ├─ VPN auto-connect
                   ├─ Dispatcher scripts
                   └─ All configuration files
```

## Further Information

- [Molecule Documentation](https://molecule.readthedocs.io/)
- [NetworkManager Documentation](https://networkmanager.dev/)
- [Vagrant Documentation](https://www.vagrantup.com/docs)
- [mac80211_hwsim Kernel Module](https://wireless.wiki.kernel.org/en/users/drivers/mac80211_hwsim)
- [marcstraube/archlinux-ansible Box](https://portal.cloud.hashicorp.com/vagrant/discover/marcstraube/archlinux-ansible)
