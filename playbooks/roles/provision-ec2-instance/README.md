# provision-ec2-instance

Provisions an EC2 instance for building VyOS AMI images.

## Description

This role creates the necessary AWS infrastructure to build a VyOS AMI:
- EC2 instance (T3.micro by default) with Debian Trixie
- Security group allowing SSH access
- SSH key pair for instance access
- Additional EBS volume for VyOS installation

## Requirements

- Ansible 2.16+
- `amazon.aws` collection >= 5.0.0
- AWS credentials configured
- boto3, botocore, boto Python packages

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# SSH key pair name
key_pair_name: vyos-build-ami

# AWS region
ec2_region: us-east-1

# Instance type (T3+ recommended for NVMe and EC2 Instance Connect)
instance_type: t3.micro

# Base Debian AMI configuration
base_image:
  debian_aws_account_id: "136693071363"
  name: "debian-13-amd64-*"
  architecture: "x86_64"
  hypervisor: "xen"
  root_device_type: "ebs"

# Volume configuration (T3 uses NVMe)
volume_size: 4
volume_drive: /dev/nvme1n1
volume_drive_partition_suffix: p1
```

## Dependencies

None.

## Example Playbook

```yaml
- hosts: local
  connection: local
  gather_facts: True
  roles:
    - provision-ec2-instance
```

## License

MIT

## Author Information

VyOS maintainers and contributors
