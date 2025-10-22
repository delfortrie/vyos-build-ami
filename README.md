# VyOS build-ami

Ansible playbooks for building VyOS AMIs on AWS EC2.

**NOTE:** You can support VyOS development by using the official VyOS AMI from the AWS marketplace: https://aws.amazon.com/marketplace/pp/B074KJK4WC

The official AMIs are built with these exact scripts so if you build one for yourself, your own AMI will be functionally identical to the official ones.

## Prerequisites

### VyOS ISO Preparation

VyOS images built with default `make iso` options *do not* include EC2 autoconfiguration mechanism.

To make an image suitable for an AMI, build with AWS support:

```bash
./configure
sudo make AWS
```

### AWS Configuration

- AWS CLI configured with credentials and default region
- Sufficient AWS permissions to create EC2 instances, AMIs, security groups, and EBS snapshots
- Available VPC and subnet in your region

## Requirements

### System Requirements

- Linux, macOS, or WSL2 on Windows
- Python 3.8 or higher
- Ansible 2.16 or higher

### Installation

1. **Set up Python virtual environment:**

```bash
./bootstrap-virtualenv
```

This will create a virtual environment in `env/`, install all dependencies, and set up Ansible collections.

2. **Activate the virtual environment:**

```bash
source env/bin/activate
```

3. **Configure AWS CLI:**

```bash
aws configure
```

See the [AWS CLI user guide](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) for detailed configuration instructions.

## Usage

### Basic Usage

```bash
./vyos-build-ami <VyOS ISO URL>
```

**Example with latest nightly build:**

```bash
./vyos-build-ami https://github.com/vyos/vyos-nightly-build/releases/download/2025.10.17-0019-rolling/vyos-2025.10.17-0019-rolling-generic-amd64.iso
```

**Finding the latest build:**

VyOS nightly builds are available at: https://vyos.net/get/nightly-builds/

Or browse releases directly: https://github.com/vyos/vyos-nightly-build/releases

### Supported Versions

- **VyOS Rolling (current branch)** - Latest nightly builds from `vyos-nightly-build` repository
- **VyOS 1.2.0 and newer** - Fully supported
- **VyOS 1.1.x** - Use the `1.1.x` tag of this repository

### ISO Requirements

VyOS ISOs for AWS must be built with EC2 support. When using official nightly builds, use the `-generic-amd64.iso` variant, which includes cloud-init and AWS autoconfiguration.

### Configuration Options

You can customize the build by modifying variables in `playbooks/group_vars/all`:

- **Region:** Default is `us-east-1`
- **Instance Type:** Default is `t3.micro` (T3+ instances support EC2 Instance Connect)
- **Volume Size:** Default is 10GB (VyOS recommended minimum)
- **Key Pair:** Default name is `vyos-build-ami`
- **SSH Access:** Default allows SSH from `0.0.0.0/0` (change `ssh_cidr_ip` in `provision-ec2-instance/defaults/main.yml`)

## How It Works

Since AWS doesn't support direct disk image uploads, this project:

1. **Launches a Debian Trixie (13) t3.micro instance** with an additional 10GB gp3 EBS volume
2. **Downloads the VyOS ISO** to `/var/tmp` on the instance
3. **Installs VyOS** to the EBS volume using overlay filesystem
4. **Configures GRUB** for EC2 serial console support
5. **Creates an EBS snapshot** of the volume
6. **Copies the snapshot with encryption** enabled (using AWS managed keys)
7. **Registers a gp3 encrypted AMI** from the encrypted snapshot with IMDSv2 required
8. **Cleans up** temporary resources

## Troubleshooting

### Failed Playbook Cleanup

If the playbook fails, the following resources may be left behind in your AWS account:

- **EC2 Instance:** t3.micro instance named "vyos-build-ami"
- **SSH Key Pair:** Named "vyos-build-ami"
- **Security Group:** Named "vyos_build_ami"
- **EBS Volume:** Attached to the instance (deleted when instance is terminated)

To manually clean up:

```bash
# Delete instance
aws ec2 terminate-instances --instance-ids <instance-id>

# Delete key pair
aws ec2 delete-key-pair --key-name vyos-build-ami

# Delete security group (after instance is terminated)
aws ec2 delete-security-group --group-name vyos_build_ami
```

### Common Issues

#### Managing Multiple AMIs

AMI names include a timestamp to prevent collisions. You can safely run the build multiple times for the same VyOS version.

To clean up old AMIs:

```bash
# List your VyOS AMIs
aws ec2 describe-images --owners self --filters "Name=name,Values=VyOS*" \
  --query 'Images[*].[ImageId,Name,CreationDate]' --output table

# De-register an old AMI
aws ec2 deregister-image --image-id <ami-id>

# Delete the associated snapshot (optional, saves storage costs)
aws ec2 delete-snapshot --snapshot-id <snapshot-id>
```

#### SSH Timeout

**Error:** Playbook times out waiting for SSH connection.

**Cause:** EC2 instance took longer than 300 seconds to boot.

**Solution:** Re-run the playbook. The existing instance will be cleaned up and a new one created.

#### T3 Instance and NVMe Storage

T3+ instances use NVMe SSD storage. The EBS volume is attached as `/dev/sdf` from AWS's API perspective, but appears as `/dev/nvme1n1` inside the instance. The playbooks handle this automatically.

For T2 instances or older instance types, you'll need to modify:
- `volume_drive: /dev/xvdf`
- `volume_drive_partition_suffix: 1`

## Architecture

### Ansible Roles

The project is organized into modular Ansible roles:

- **provision-ec2-instance** - Creates build instance, security group, and SSH key pair
- **build-disk** - Downloads ISO, installs VyOS to EBS volume, configures GRUB
- **build-vyos-ami** - Creates snapshot and registers AMI
- **ec2-cleanup** - Removes temporary resources

### Technology Stack

- **Base Instance:** Debian 13 (Trixie) on T3.micro
- **Storage:** gp3 EBS volumes with NVMe interface, encrypted at rest
- **Filesystem:** Overlay (modern replacement for AUFS)
- **Boot Loader:** GRUB2 with serial console support
- **Ansible Collections:** `amazon.aws` (>= 5.0.0)
- **Ansible Version:** 9.x-11.x (ansible-core 2.16-2.19)

### Security

- **IMDSv2:** Required for both build instances and created AMIs
- **Encryption:** EBS volumes encrypted using AWS managed keys
- **SSH:** Password authentication disabled, key-based only
- **EC2 Instance Connect:** Supported on T3+ instances
- **Network:** Security group allows SSH from configurable CIDR (default 0.0.0.0/0, customize via `ssh_cidr_ip`)
- **Temporary Resources:** Build infrastructure automatically cleaned up after AMI creation

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test with a real VyOS ISO build
5. Submit a pull request

### Testing

Before submitting changes, test with:

```bash
./vyos-build-ami <test-iso-url>
```

Verify the resulting AMI boots and is accessible via SSH.

## License

These scripts are available under the MIT license. See the LICENSE file for more info.

build-ami playbooks were originally written by hydrajump (https://github.com/hydrajump) and are now maintained
by the VyOS team.
