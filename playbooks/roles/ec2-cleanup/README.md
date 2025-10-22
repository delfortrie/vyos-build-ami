# ec2-cleanup

Removes temporary resources created during AMI build process.

## Description

This role cleans up AWS resources created during the VyOS AMI build:
- Terminates the build EC2 instance
- Deletes the SSH key pair
- Removes the security group
- Deletes temporary SSH key files

## Requirements

- Ansible 2.16+
- `amazon.aws` collection >= 5.0.0
- AWS credentials configured

## Role Variables

Variables are inherited from `provision-ec2-instance` role:

```yaml
# AWS region
ec2_region: us-east-1

# SSH key pair name
key_pair_name: vyos-build-ami

# Temporary folder for SSH keys
temp_folder: "{{ playbook_dir }}/files/ssh-keys"

# Security group ID (from provision role)
security_group.group_id: <dynamically created>

# EC2 instance info (from provision role)
ec2_instance.instances: <dynamically created>
```

## Dependencies

- `provision-ec2-instance` role (provides security_group variable)

## Error Handling

This role uses `ignore_errors: true` for most tasks to ensure cleanup continues even if resources were already manually deleted or don't exist.

## Manual Cleanup

If automated cleanup fails, you can manually remove resources:

```bash
# Find and terminate instances
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=vyos-build-ami" "Name=tag:Type,Values=VyOS" \
  --query 'Reservations[*].Instances[*].InstanceId' --output text

aws ec2 terminate-instances --instance-ids <instance-id>

# Delete key pair
aws ec2 delete-key-pair --key-name vyos-build-ami

# Delete security group (after instance termination)
aws ec2 delete-security-group --group-name vyos_build_ami
```

## License

MIT

## Author Information

VyOS maintainers and contributors
