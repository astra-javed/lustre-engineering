# LAB-008 — Pi5 Identity and Storage Baseline

**Date:** 2026-09-24
**Node:** `p5.lab.astrahm.com`
**Short hostname:** `p5`
**Lab IP:** `10.10.10.20/24`
**Lab gateway:** `10.10.10.1`
**Repository:** `lustre-engineering`
**Branch at documentation checkpoint:** `main`

---

## 1. Objective

Establish and document the Raspberry Pi 5 identity, network configuration, storage inventory, filesystem state, and baseline verification required for its role in the AstraHM home lab.

This LAB was performed as a controlled discovery and configuration exercise.

No partitioning, formatting, filesystem creation, or mounting of the secondary NVMe device was performed.

---

## 2. WHY

The Pi5 needs a deterministic identity and network path before it is used for storage and Lustre engineering work.

The storage baseline is also required before making any decisions about the unused capacity on the OS NVMe or the existing XFS filesystem on the secondary NVMe.

The engineering workflow used was:

**WHY → DESIGN → CHANGE → VERIFY → SAVE → DOCUMENT → GIT**

---

## 3. Hardware Identity

### Raspberry Pi

- Board: `Raspberry Pi 5 Model B Rev 1.1`
- Device-tree compatible:
  - `raspberrypi,5-model-b`
  - `brcm,bcm2712`
- Architecture: `aarch64` / `arm64`
- CPU: 4 × ARM Cortex-A76
- CPU range observed: 1500–2400 MHz
- RAM: approximately 4 GiB

### Operating System

- OS: Rocky Linux 9.8 (Blue Onyx)
- Kernel: `6.12.80-2.el9.altarch.aarch64+16k`
- Static hostname before LAB-008: unset
- Transient hostname before LAB-008: `localhost`

---

## 4. Final Identity

The Pi5 identity was changed to:

```text
Short hostname: p5
FQDN:           p5.lab.astrahm.com
IPv4 address:   10.10.10.20/24
Gateway:        10.10.10.1
DNS:            10.10.10.1
```

Hostname was set with:

```bash
hostnamectl set-hostname p5.lab.astrahm.com
```

Verification:

```bash
hostname -s
```

Result:

```text
p5
```

`hostnamectl` confirmed:

```text
Static hostname: p5.lab.astrahm.com
```

---

## 5. Network Design

The intended Pi5 network path is:

```text
AstraHM Pi5
    |
    | eth0
    | 10.10.10.20/24
    |
TP-Link sw01
    |
EdgeRouter erx01
    | 10.10.10.1
    |
Airtel WAN
```

The Pi5 previously had both Ethernet and Wi-Fi active.

Before the change:

```text
eth0  → 10.10.10.20/24
wlan0 → 192.168.0.45/24
```

The Ethernet default route had metric 100 and the Wi-Fi default route had metric 600.

For a deterministic server/lab node, Ethernet was retained as the active lab path.

---

## 6. Wi-Fi Configuration Change

The NetworkManager Wi-Fi profile was:

```text
Profile: MAARIYA
Interface: wlan0
```

Wi-Fi autoconnect was disabled:

```bash
nmcli connection modify "MAARIYA" connection.autoconnect no
```

The active Wi-Fi connection was then disconnected:

```bash
nmcli connection down "MAARIYA"
```

Final interface state:

```text
eth0   UP    10.10.10.20/24
wlan0  DOWN
```

Final routing table:

```text
default via 10.10.10.1 dev eth0 proto static metric 100
10.10.10.0/24 dev eth0 proto kernel scope link src 10.10.10.20 metric 100
```

The Pi5 therefore has one active IPv4 default path through the AstraHM LAN.

---

## 7. NetworkManager Ethernet Profile

Ethernet profile:

```text
Name: Wired connection 1
UUID: 4689894a-5af4-3be8-95d0-5c5f822f410f
Interface: eth0
Autoconnect: yes
IPv4 address: 10.10.10.20/24
Gateway: 10.10.10.1
DNS: 10.10.10.1
Default route: yes
Route metric: 100
```

---

## 8. Connectivity Verification

### IPv4 Internet

Command:

```bash
ping -c 3 8.8.8.8
```

Result:

```text
3 packets transmitted
3 packets received
0% packet loss
Average RTT: 9.540 ms
```

### IPv4 DNS

Command:

```bash
getent ahostsv4 google.com
```

Result included:

```text
142.250.207.142 google.com
```

### Pi5 hostname resolution

The Pi5 was initially given a temporary `/etc/hosts` entry for local identity testing:

```text
10.10.10.20   p5.lab.astrahm.com p5
```

After the EdgeRouter DNS record was created and verified, the temporary entry was removed.

Final `/etc/hosts` contains only the standard localhost entries.

---

## 9. EdgeRouter DNS Record

The AstraHM EdgeRouter was updated with:

```text
p5.lab.astrahm.com → 10.10.10.20
```

Configuration command:

```text
set system static-host-mapping host-name p5.lab.astrahm.com inet 10.10.10.20
```

The candidate configuration was reviewed with:

```text
compare
```

The change was committed and then persisted with:

```text
commit
save
```

Verification on the EdgeRouter:

```text
set system static-host-mapping host-name p5.lab.astrahm.com inet 10.10.10.20
```

The EdgeRouter configuration was saved to:

```text
/config/config.boot
```

---

## 10. DNS Verification from dev-box-01

`dev-box-01` uses:

```text
DNS: 10.10.10.1
```

Before the EdgeRouter DNS record existed:

```bash
getent hosts p5.lab.astrahm.com
```

returned no result.

After the EdgeRouter change:

```text
10.10.10.20     p5.lab.astrahm.com
```

This confirmed DNS resolution through the lab DNS path.

---

## 11. SSH Verification

SSH was verified using the new FQDN:

```bash
ssh root@p5.lab.astrahm.com
```

The hostname resolved to:

```text
10.10.10.20
```

The SSH ED25519 host key was accepted and added to the `dev-box-01` known-hosts database.

Successful login prompt:

```text
[root@p5 ~]#
```

This confirms administration through the new lab FQDN.

---

# 12. NVMe Hardware Inventory

The Pi5 contains two NVMe devices.

## NVMe Controller 0

```text
Device:       /dev/nvme0n1
Model:        Micron 2200S NVMe 512GB
Serial:       2020282FFE91
Firmware:     22001070
State:        live
```

## NVMe Controller 1

```text
Device:       /dev/nvme1n1
Model:        KXG60ZNV512G NVMe KIOXIA 512GB
Serial:       40OF70MDF7HL
Firmware:     10604106
State:        live
```

Both controllers were confirmed by Linux sysfs to be in the `live` state.

---

# 13. PCIe Topology

The Pi5 PCIe inventory showed:

```text
Broadcom BCM2712 PCIe Bridge
    |
    +-- ASMedia ASM1182e PCIe switch
         |
         +-- Micron 2200S NVMe
         |
         +-- Toshiba/KIOXIA NVMe
```

The relevant kernel messages reported the NVMe controllers behind a PCIe 5.0 GT/s x1 upstream link.

The second BCM2712 PCIe hierarchy contained the Raspberry Pi RP1 South Bridge.

No obvious NVMe or PCIe errors were observed in the captured boot messages.

---

# 14. Micron OS NVMe Layout

Physical disk:

```text
/dev/nvme0n1
Size: 476.94 GiB
Model: Micron 2200S NVMe 512GB
Partition table: DOS/MBR
```

Partition layout:

```text
/dev/nvme0n1p1   500M   W95 FAT32 (LBA)   /boot/efi
/dev/nvme0n1p2   512M   Linux swap        [SWAP]
/dev/nvme0n1p3   2.6G   Linux             /
```

The remaining approximately 474 GiB of the physical disk is currently unallocated.

No partitioning operation was performed during LAB-008.

---

# 15. Root Filesystem

Root filesystem:

```text
Device: /dev/nvme0n1p3
Filesystem: ext4
Size: approximately 2.5 GiB
Used: approximately 1.8 GiB
Available: approximately 541 MiB
Usage: approximately 78%
```

Directory usage observed:

```text
/usr    1.7G
/var    155M
/etc     23M
/boot   488K
/root    36K
/home    28K
/tmp     36K
```

The small root filesystem is therefore primarily occupied by the Rocky Linux userspace under `/usr`.

No root filesystem resize was performed.

---

# 16. KIOXIA NVMe Layout

Physical disk:

```text
/dev/nvme1n1
Size: 476.94 GiB
Model: KXG60ZNV512G NVMe KIOXIA 512GB
Partition table: GPT
```

Partition:

```text
/dev/nvme1n1p1
Size: 476.9G
Type: Linux filesystem
Filesystem: XFS
Label: ASTRA-NVME1
UUID: 320f4881-ec2a-4a51-8823-a476dc99f7f1
```

The partition is currently **not mounted**.

No formatting or mounting was performed during LAB-008.

---

# 17. KIOXIA XFS Geometry

`xfs_info` reported:

```text
block size:       4096
allocation groups: 4
inode size:       512
metadata CRC:     enabled
finobt:            enabled
rmapbt:            enabled
reflink:           enabled
bigtime:           enabled
inobtcount:        enabled
nrext64:           enabled
internal log:      yes
realtime device:   none
```

Filesystem superblock values:

```text
blocksize = 4096
dblocks   = 125026560
agcount   = 4
fdblocks  = 124965476
```

The difference between total and free data blocks was:

```text
125026560 - 124965476 = 61084 blocks
```

At 4 KiB per block, this is approximately 238.6 MiB of allocated filesystem blocks.

The filesystem is therefore essentially unused at this baseline point.

---

# 18. XFS Integrity Check

A read-only XFS repair check was performed:

```bash
xfs_repair -n /dev/nvme1n1p1
```

The check completed through the applicable verification phases:

- Phase 1 — superblock verification
- Phase 2 — internal log and filesystem maps
- Phase 3 — allocation-group inode discovery
- Phase 4 — duplicate block checking
- Phase 6 — inode connectivity
- Phase 7 — link-count verification

The command reported:

```text
No modify flag set
```

No filesystem modification was performed.

---

# 19. XFS Identity Verification

The XFS superblock was queried read-only:

```bash
xfs_db -r -c 'sb 0' -c 'p uuid fname' /dev/nvme1n1p1
```

Result:

```text
uuid = 320f4881-ec2a-4a51-8823-a476dc99f7f1
fname = "ASTRA-NVME1"
```

This matches the earlier `blkid` result.

---

# 20. Boot Configuration Baseline

The Pi5 kernel command line included:

```text
root=LABEL=RPIROOT
rootfstype=ext4
rootwait
nvme.max_host_mem_size_mb=0
pci=pcie_bus_safe
```

The root filesystem is therefore selected by filesystem label:

```text
RPIROOT
```

which corresponds to:

```text
/dev/nvme0n1p3
```

No boot parameters were modified during LAB-008.

---

# 21. Current Mount State

Important mounted filesystems:

```text
/          /dev/nvme0n1p3  ext4
/boot/efi  /dev/nvme0n1p1  vfat
```

The secondary filesystem remains unmounted:

```text
/dev/nvme1n1p1  XFS  ASTRA-NVME1  not mounted
```

---

# 22. Storage Design Decision

No storage reconfiguration is being performed as part of LAB-008.

Current intended roles:

```text
Micron 512GB
    └── Rocky Linux OS
        └── Existing EFI + swap + 2.6G root
        └── ~474G intentionally left unallocated

KIOXIA 512GB
    └── Existing XFS filesystem
        └── ASTRA-NVME1
        └── Currently unmounted
```

The unallocated Micron capacity is intentionally preserved for a future storage design decision.

The KIOXIA XFS filesystem is intentionally preserved without mounting or reformatting.

Future work may use these devices for controlled storage experiments, benchmarking, filesystem studies, RAID/erasure-coding experiments, or other AstraHM/Lustre engineering work. No such design is committed by LAB-008.

---

# 23. Changes Made in LAB-008

### Pi5

1. Set static hostname:
   ```text
   p5.lab.astrahm.com
   ```

2. Disabled Wi-Fi autoconnect:
   ```text
   MAARIYA → connection.autoconnect=no
   ```

3. Disconnected active Wi-Fi:
   ```text
   wlan0 → DOWN
   ```

4. Added a temporary local `/etc/hosts` entry for testing.

5. Removed the temporary `/etc/hosts` entry after lab DNS was established.

### EdgeRouter

Added and committed:

```text
p5.lab.astrahm.com → 10.10.10.20
```

Configuration was saved to `/config/config.boot`.

### Storage

No partitioning, formatting, mounting, resizing, or filesystem modification was performed.

---

# 24. Verification Summary

| Verification | Result |
|---|---|
| Pi5 hostname | PASS |
| Short hostname `p5` | PASS |
| Ethernet `10.10.10.20/24` | PASS |
| Default route via `10.10.10.1` | PASS |
| Wi-Fi disconnected | PASS |
| Wi-Fi autoconnect disabled | PASS |
| IPv4 Internet connectivity | PASS |
| IPv4 DNS resolution | PASS |
| EdgeRouter DNS record | PASS |
| dev-box DNS resolution | PASS |
| SSH via FQDN | PASS |
| Both NVMe controllers detected | PASS |
| Both NVMe controllers live | PASS |
| Micron partition layout captured | PASS |
| KIOXIA partition layout captured | PASS |
| XFS geometry captured | PASS |
| XFS read-only repair check | PASS |
| KIOXIA remains unmounted | PASS |
| Micron free space preserved | PASS |

---

# 25. Final Pi5 State

```text
Hostname:
    p5.lab.astrahm.com

IP:
    10.10.10.20/24

Gateway:
    10.10.10.1

DNS:
    10.10.10.1

Primary network interface:
    eth0

Wi-Fi:
    wlan0 disconnected
    autoconnect disabled

OS:
    Rocky Linux 9.8

Kernel:
    6.12.80-2.el9.altarch.aarch64+16k

Hardware:
    Raspberry Pi 5 Model B Rev 1.1
    Broadcom BCM2712
    4 × Cortex-A76
    ~4 GiB RAM

Storage:
    Micron 2200S 512GB
        OS disk
        ~474 GiB unallocated

    KIOXIA KXG60ZNV512G 512GB
        XFS
        ASTRA-NVME1
        unmounted
```

---

# 26. Next Steps

LAB-008 does not resize, repartition, reformat, or mount storage.

Potential follow-up work:

1. Decide how to use the ~474 GiB unallocated Micron capacity.
2. Decide the future role of `ASTRA-NVME1`.
3. Establish SSH-key authentication for the Pi5.
4. Review and harden root SSH access.
5. Continue Pi5 storage/performance baseline.
6. Add the Pi5 to the lab inventory and Ansible inventory.
7. Begin controlled storage benchmarking only after the storage design is approved.

---

## LAB-008 Status

**Status: Baseline and identity configuration complete; documentation ready for review.**

No Git commit is included in this document. Review this file first, then commit it to `lustre-engineering` after approval.
