# build-vyos-ami

Creates an EBS snapshot and registers it as an AMI.

## Description

This role finalizes the AMI creation process:
- Creates an EBS snapshot of the VyOS-installed volume
- Stops the build instance
- Registers an AMI from the snapshot
- Tags the AMI with build metadata

## Requirements

- Ansible 2.16+
- `amazon.aws` collection >= 5.0.0
- AWS credentials configured
- Completed `build-disk` role execution

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml` and `vars/main.yml`):

```yaml
# AWS region
ec2_region: us-east-1

# EBS drive to snapshot
ebs_drive: /dev/sdf

# AMI configuration (derived from VyOS version)
ami_name: VyOS (HVM) {{ version_string }}
ami_description: The VyOS AMI is an EBS-backed, HVM image...
ami_architecture: x86_64
ami_virtualization_type: hvm
ami_root_device_name: /dev/xvda
```

## Dependencies

- `provision-ec2-instance` role
- `build-disk` role

## AMI Features

The registered AMI includes:
- ENA (Elastic Network Adapter) support
- SR-IOV networking support
- HVM virtualization
- EBS-backed storage
- Serial console access
- EC2 Instance Connect support (T3+ instances)

## Tags

AMIs are tagged with:
- `iso_url` - Source ISO URL
- `iso` - ISO filename
- `build_user` - User who ran the build
- `build_controller_host` - Hostname of build controller

## License

MIT

## Author Information

VyOS maintainers and contributors
