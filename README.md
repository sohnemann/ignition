# Fedora CoreOS PXE Boot Setup

This repository contains configuration files for deploying Fedora CoreOS via PXE boot with iPXE.

## Configuration Files

- `butane.bu` - Butane configuration (human-readable YAML format)
- `ignition.ign` - Ignition configuration (generated from Butane, used by CoreOS)

## Setup Overview

The configuration provisions a Fedora CoreOS system with:

- SSH access for `core` user
- Persistent `/var` storage using btrfs with compression
- Hostname set to `tui`
- Automatic rebase to uCore (Universal Blue's minimal CoreOS variant)

### Storage Configuration

The system uses FCOS's default partition layout with a btrfs subvolume for `/var`:

- Mounts `/var` as a subvolume on the root partition (`/dev/disk/by-partlabel/root`)
- Enables zstd compression for efficient storage usage
- Provides persistent storage for containers and logs

## Deployment Steps

1. **Write Butane configuration** (`butane.bu`)
   - Configure users, SSH keys, storage, systemd units

2. **Convert Butane to Ignition format**

   ```bash
   docker run --interactive --rm --security-opt label=disable \
     --volume ${PWD}:/pwd --workdir /pwd \
     quay.io/coreos/butane:release --pretty --strict butane.bu > ignition.ign
   ```

3. **Host CoreOS files via HTTP server** (e.g., on DiskStation)

   ```text
   fedora-coreos-42.20250914.3.0-live-initramfs.x86_64.img
   fedora-coreos-42.20250914.3.0-live-kernel.x86_64
   fedora-coreos-42.20250914.3.0-live-rootfs.x86_64.img
   ignition.ign
   ```

4. **Create iPXE embed script** pointing to CoreOS files and Ignition config

5. **Clone iPXE repository**

   ```bash
   git clone https://github.com/ipxe/ipxe.git
   ```

6. **Build iPXE with embedded script**

   ```bash
   cd ipxe/src
   make bin-x86_64-efi/ipxe.efi EMBED=myscript.ipxe
   ```

7. **Deploy iPXE bootloader**
   - Copy `ipxe.efi` to TFTP server

8. **Configure DHCP server**
   - Point PXE clients to `ipxe.efi`

9. **Boot target system**
   - Start host and select PXE boot from network

## Post-Installation

After first boot, the system will:

1. Rebase to unsigned uCore OCI image
2. Reboot
3. Rebase to signed uCore OCI image
4. Reboot into final configuration

SSH access will be available on subsequent boots using the configured SSH key.
