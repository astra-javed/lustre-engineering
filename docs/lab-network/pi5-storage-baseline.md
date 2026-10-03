# Pi 5 Storage Baseline

**Lab:** AstraHM / Lustre Engineering Lab
**Host:** p5.lab.astrahm.com
**IP:** 10.10.10.20
**Date:** 2026-10-03
**Status:** Baseline captured before iSCSI target implementation

## 1. Purpose

Document the current Raspberry Pi 5 storage and kernel state before implementing remote block storage for the AstraHM ExaScaler 6.3 lab.

The Pi 5 is planned to provide remote block storage to ExaScaler virtual machines running on the Proxmox host.

Planned architecture:

    Pi 5
     |
     +-- KIOXIA NVMe 512 GB
           |
           +-- MDT storage
           +-- OST01 storage
           +-- OST02 storage
           +-- optional test storage
                  |
                  v
           ExaScaler VMs

The current iSCSI target implementation is not yet active.

## 2. Operating System

    OS:       Rocky Linux 9.8 (Blue Onyx)
    Arch:     aarch64
    Kernel:   6.12.80-2.el9.altarch.aarch64+16k

The Pi 5 is using the Rocky Linux Raspberry Pi kernel.

## 3. NVMe Storage

### System NVMe

    Device:  /dev/nvme0n1
    Model:   Micron 2200S NVMe 512GB
    Serial:  2020282FFE91
    Size:    approximately 477 GiB

Current partitions:

    /dev/nvme0n1p1   500M   vfat   /boot/efi
    /dev/nvme0n1p2   512M   swap
    /dev/nvme0n1p3   ~477G  ext4   /

The root filesystem was originally approximately 2.6 GiB and was expanded to use the available capacity.

After expansion:

    Filesystem: /dev/nvme0n1p3
    Filesystem type: ext4
    Size: approximately 469G
    Mount: /
    Usage: approximately 1%

The expansion was performed using:

    growpart
    resize2fs

### Storage NVMe

    Device:  /dev/nvme1n1
    Model:   KXG60ZNV512G KIOXIA 512GB
    Serial:  40OF70MDF7HL
    Size:    approximately 477 GiB

Partition:

    /dev/nvme1n1p1
    Filesystem: XFS
    Label:      ASTRA-NVME1
    UUID:       320f4881-ec2a-4a51-8823-a476dc99f7f1

The filesystem was temporarily mounted read-only for verification.

The filesystem was confirmed to be empty and was subsequently unmounted.

Current state:

    /dev/nvme1n1p1
        |
        +-- XFS filesystem exists
        +-- currently unmounted
        +-- no production data
        +-- candidate storage for lab iSCSI implementation

No repartitioning or destructive formatting has been performed.

## 4. iSCSI Target Investigation

targetcli is installed:

    targetcli-2.1.57-3.el9.noarch

However:

    targetcli ls

returns:

    Could not create RTSRoot in configFS

The configfs filesystem itself is mounted at:

    /sys/kernel/config

The running kernel provides SCSI and iSCSI initiator functionality:

    CONFIG_SCSI_MOD=y
    CONFIG_SCSI_COMMON=y
    CONFIG_SCSI=y
    CONFIG_SCSI_DMA=y
    CONFIG_SCSI_ISCSI_ATTRS=y
    CONFIG_SCSI_LOWLEVEL=y
    CONFIG_ISCSI_TCP=m
    CONFIG_ISCSI_BOOT_SYSFS=m

However, CONFIG_TARGET_CORE is not enabled.

Therefore the current kernel provides iSCSI initiator functionality but does not provide the LIO SCSI target subsystem required by targetcli.

## 5. Kernel Investigation

Current running kernel:

    6.12.80-2.el9.altarch.aarch64+16k

Available Rocky Raspberry Pi kernel:

    6.12.103-1.el9.altarch.aarch64+16k

The available 6.12.103 kernel package contains its kernel configuration:

    /boot/config-6.12.103-1.el9.altarch.aarch64+16k

The configuration was inspected directly from the downloaded RPM without installing the kernel.

No TARGET or ISCSI configuration entries were found in the 6.12.103 Raspberry Pi kernel configuration.

Therefore, upgrading the Rocky Raspberry Pi kernel alone is not currently expected to provide the required LIO target functionality.

Rocky also provides metadata for the Raspberry Pi kernel source package:

    kernel-rpi-6.12.103-1.el9.altarch.src.rpm

The source RPM was not successfully downloaded through the currently configured source repositories.

No kernel replacement or reboot has been performed.

## 6. Standard Rocky ARM64 Kernel Investigation

The standard Rocky ARM64 kernel packages were inspected for comparison.

The standard ARM64 kernel modules provide LIO components including:

    target_core_mod.ko
    target_core_file.ko
    target_core_iblock.ko
    target_core_pscsi.ko
    iscsi_target_mod.ko

This demonstrates that Rocky's standard ARM64 kernel packaging contains the required Linux SCSI target functionality.

However, the standard ARM64 kernel package did not show an obvious Raspberry Pi 5 / BCM2712 device-tree entry during the investigation.

Therefore, the standard ARM64 kernel has not been installed or booted on the Pi 5.

## 7. Current Architecture

The Pi 5 remains on its known-good Rocky Raspberry Pi kernel.

No kernel changes have been made.

The KIOXIA NVMe remains available as the future storage backend.

The iSCSI target implementation is still under investigation.

Planned architecture:

    TP-Link Switch
          |
         Pi5
          |
    KIOXIA NVMe
          |
     iSCSI Target
          |
    +-----+-----+-----+
    |           |     |
    |           |     |
 ExaVM01     ExaVM02 ExaVM03
 MDS/MDT      OSS01   OSS02

Multipath will be introduced later when the network and storage path design is ready.

## 8. Safety State

The following have NOT been performed:

- No destructive formatting of the KIOXIA storage
- No repartitioning of the KIOXIA storage
- No replacement of the running Pi kernel
- No kernel reboot
- No iSCSI target configuration
- No ExaScaler storage LUN creation

The existing Rocky Linux Pi environment remains the recovery baseline.

## 9. Next Investigation

Before modifying the Pi kernel, determine the safest method to provide an iSCSI target with Pi 5 hardware support.

Candidate approaches:

1. Build a Pi 5-compatible Rocky kernel with LIO enabled.
2. Determine whether an existing Pi-compatible kernel/package provides LIO.
3. Evaluate an alternative block-storage export mechanism if LIO cannot be provided reliably.

The final solution must preserve reliable Pi 5 boot capability and support the ExaScaler 6.3 storage architecture.

## 10. Engineering Workflow

All infrastructure changes follow:

    WHY
      |
    DESIGN
      |
    CHANGE
      |
    VERIFY
      |
    COMMIT WITH MESSAGE
      |
    SAVE
      |
    DOCUMENT
      |
    GIT PUSH

No kernel or storage-destructive change should be performed without verification and a recovery path.

---

**Status:** Pi 5 storage baseline documented.

**Next:** Decide and validate the iSCSI target implementation path.
