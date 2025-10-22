# build-disk

Installs VyOS from ISO to an EBS volume on the build instance.

## Description

This role performs the VyOS installation process:
- Downloads VyOS ISO to the build instance
- Mounts and verifies the ISO image
- Partitions the attached EBS volume
- Installs VyOS to the volume using overlay filesystem
- Configures GRUB bootloader with EC2 serial console support
- Applies default EC2 configuration for VyOS

## Requirements

- Ansible 2.16+
- Running on a Debian-based EC2 instance
- Additional EBS volume attached
- VyOS ISO URL provided

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# VyOS ISO URL (passed from command line)
vyos_iso_url: "{{ iso }}"
vyos_iso_local: /tmp/vyos.iso

# Filesystem configuration
ROOT_FSTYPE: ext4
ROOT_PARTITION: "{{ volume_drive }}{{ volume_drive_partition_suffix }}"

# Volume configuration (inherited from provision role)
volume_size: 4
volume_drive: /dev/nvme1n1
volume_drive_partition_suffix: p1

# Mount points for installation
CD_ROOT: /mnt/cdrom
CD_SQUASH_ROOT: /mnt/cdsquash
WRITE_ROOT: /mnt/wroot
READ_ROOT: /mnt/squashfs
INSTALL_ROOT: /mnt/inst_root
```

## Dependencies

- `provision-ec2-instance` role (must run first)

## How It Works

1. Downloads VyOS ISO and verifies checksums
2. Mounts ISO and squashfs filesystem
3. Partitions EBS volume with MSDOS partition table
4. Creates ext4 filesystem with persistence label
5. Copies VyOS files to volume
6. Sets up overlay filesystem for installation
7. Installs GRUB to volume's boot sector
8. Configures GRUB for EC2 serial console
9. Applies default VyOS EC2 configuration
10. Unmounts all filesystems

## Templates

- `boot/grub/grub.cfg.j2` - GRUB configuration with serial console support
- `boot/grub/device.map.j2` - GRUB device mapping
- `config.boot.default.ec2` - Default VyOS configuration for EC2
- `persistence.conf` - Persistence configuration

## License

MIT

## Author Information

VyOS maintainers and contributors
