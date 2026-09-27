# ANSIBLE-004 — Proxmox OSS Data Disk Provisioning

## Objective

Provision a dedicated 100 GiB data disk for the Lustre OSS VM using Ansible and the Proxmox API.

The disk is intentionally left unformatted and unmounted. Filesystem creation will be handled later as part of the Lustre OSS storage configuration.

## Environment

| Item | Value |
|---|---|
| Proxmox host | pve.lab.astrahm.com |
| Proxmox IP | 10.10.10.10 |
| VM ID | 1003 |
| VM hostname | lustre-oss01.lab.astrahm.com |
| VM role | Lustre OSS |
| Existing OS disk | 32 GiB |
| New data disk | 100 GiB |
| Proxmox storage | local-lvm |
| Proxmox disk slot | scsi1 |
| Guest device | /dev/sdb |

## Design

The OSS VM has two storage roles:

    scsi0
      |
      +-- 32 GiB
      +-- Rocky Linux OS
      +-- /dev/sda

    scsi1
      |
      +-- 100 GiB
      +-- Lustre OSS data
      +-- /dev/sdb

The operating-system disk remains separate from the future Lustre filesystem storage.

The new disk is not formatted at this stage because the filesystem and storage layout will be decided during the Lustre OSS provisioning phase.

## Ansible Automation

The disk is provisioned by:

    ansible/playbooks/02-proxmox-oss-disk.yml

The playbook uses the community.proxmox collection and the proxmox_disk module.

Configured values:

    oss_vmid: 1003
    oss_data_disk: scsi1
    oss_data_storage: local-lvm
    oss_data_size_gib: 100

The playbook creates the disk through the Proxmox API rather than using the Proxmox CLI manually.

## Proxmox Authorization

The Ansible automation uses the Proxmox account:

    ansible@pam

and API token:

    ansible@pam!lab-automation

The token secret is stored in Ansible Vault and is not documented here.

The automation account requires permissions for both the VM and the backing Proxmox storage.

VM permission:

    /vms/1003
    PVEVMAdmin

Storage permission:

    /storage/local-lvm
    PVEDatastoreUser

## Troubleshooting

The first execution of the playbook failed with:

    403 Forbidden: Permission check failed
    (/storage/local-lvm, Datastore.AllocateSpace)

The API authentication was working correctly.

The failure occurred because the automation account did not have permission to allocate space from the local-lvm datastore.

The required PVEDatastoreUser permission was added to both the backing automation user and the privilege-separated API token.

The playbook was then executed again successfully.

## Execution Result

The final Ansible execution completed with:

    changed=1
    unreachable=0
    failed=0

Proxmox VM configuration now contains:

    scsi0: local-lvm:vm-1003-disk-0,iothread=1,size=32G
    scsi1: local-lvm:vm-1003-disk-1,size=100G

## Guest Verification

The OSS guest was verified using Ansible.

The resulting storage layout includes:

    sda   32G   OS disk
    sdb  100G   new data disk

The new disk is visible inside the guest as:

    /dev/sdb

A stable device identifier is also available:

    /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_drive-scsi1

For future storage automation, the stable /dev/disk/by-id path should be preferred over assuming that /dev/sdb will always remain unchanged.

## Filesystem Status

The new 100 GiB disk is currently:

    unformatted
    unmounted
    unused by the operating system

No filesystem was created during this automation step.

This is intentional because the disk will later be used for Lustre OSS storage.

## MDS Status

The MDS VM is:

    lustre-mds01.lab.astrahm.com
    VM ID 1002

The MDS currently has only its 32 GiB operating-system disk.

No dedicated MDS data disk has been provisioned yet.

## Final State

    Proxmox
      |
      +-- VM 1003: lustre-oss01.lab.astrahm.com
            |
            +-- scsi0 -> 32 GiB OS disk
            |
            +-- scsi1 -> 100 GiB Lustre data disk

Inside the guest:

    /dev/sda -> Rocky Linux OS
    /dev/sdb -> reserved for Lustre OSS storage

## Result

ANSIBLE-004 successfully provisions the dedicated 100 GiB OSS data disk through the Proxmox API using Ansible.

The infrastructure layer is ready for the next Lustre storage-provisioning stage.

The disk remains unformatted so that the Lustre filesystem design can be applied explicitly in the next stage.

## Follow-up

1. Automate the required Proxmox permissions.
2. Prepare the OSS data disk for Lustre storage.
3. Determine the appropriate ldiskfs filesystem layout.
4. Install and configure Lustre OSS components.
5. Verify the filesystem from the MDS and client.
