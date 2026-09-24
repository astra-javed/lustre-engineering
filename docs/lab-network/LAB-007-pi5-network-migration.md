# LAB-007 — Pi5 Network Migration

Date: 2026-09-24
Status: Complete
Lab: AstraHM Home Lab

## 1. Objective

Migrate Raspberry Pi 5 (p5) from the Airtel LAN to the AstraHM internal LAN.

### Target Configuration

| Parameter | Value |
|---|---|
| Hostname | p5 |
| Interface | eth0 |
| IPv4 Address | 10.10.10.20/24 |
| Gateway | 10.10.10.1 |
| DNS | 10.10.10.1 |
| Network | 10.10.10.0/24 |
| Switch | TP-Link sw01 |

## 2. Initial State

The Pi5 was initially reachable through Airtel Wi-Fi.

### Wi-Fi

Interface: wlan0
SSID: MAARIYA
IPv4: 192.168.0.45/24
Gateway: 192.168.0.1

### Ethernet

Interface: eth0
MAC: 98:fe:54:15:9a:ee
Profile: Wired connection 1

The Ethernet NetworkManager profile initially used automatic IPv4 configuration.

Wi-Fi was kept available as the management and recovery path during the migration.

## 3. AstraHM Network Design

The AstraHM LAN uses:

Network: 10.10.10.0/24
Gateway: 10.10.10.1
DNS: 10.10.10.1

The Pi5 was assigned:

Host: p5
IP: 10.10.10.20/24

### Physical Path

Pi5 eth0
  |
  v
TP-Link sw01
  |
  v
EdgeRouter erx01
10.10.10.1
  |
  v
Airtel Router
192.168.0.1
  |
  v
Internet

## 4. NetworkManager Configuration

The existing profile was used:

Wired connection 1

The profile was changed from automatic DHCP to static IPv4 configuration.

Configured values:

ipv4.method: manual
ipv4.addresses: 10.10.10.20/24
ipv4.gateway: 10.10.10.1
ipv4.dns: 10.10.10.1

The profile was deactivated and reactivated to apply the configuration.

## 5. Physical Migration

After the Ethernet configuration was prepared, the Pi5 was moved to its permanent shelf location.

The Ethernet cable was connected from the Pi5 to the TP-Link sw01.

The Ethernet interface reported UP and LOWER_UP, confirming the physical Ethernet link was active.

## 6. Verification

### 6.1 IP Address

eth0 was verified with:

10.10.10.20/24

### 6.2 AstraHM Gateway

Command:

ping -c 3 10.10.10.1

Result:

2 packets transmitted
2 packets received
0% packet loss

The AstraHM gateway was reachable.

### 6.3 Internet Connectivity

Command:

ping -c 3 8.8.8.8

Result:

3 packets transmitted
3 packets received
0% packet loss

Observed latency was approximately 9 ms.

### 6.4 DNS Resolution

Command:

getent hosts google.com

DNS successfully resolved google.com.

### 6.5 Remote SSH

From the Mac:

ssh root@10.10.10.20

SSH successfully connected through the AstraHM LAN.

## 7. SSH Administration

Root password SSH access was enabled during LAB-007.

Effective configuration:

PermitRootLogin yes
PasswordAuthentication yes

This is currently being used for home-lab administration.

Future hardening should migrate to SSH key authentication and disable root/password SSH.

## 8. Final Configuration

Hostname: p5
Interface: eth0
IPv4: 10.10.10.20/24
Gateway: 10.10.10.1
DNS: 10.10.10.1

The Pi5 is now connected to the AstraHM internal network.

## 9. Verification Summary

| Test | Result |
|---|---|
| Ethernet physical link | PASS |
| Static IP 10.10.10.20/24 | PASS |
| Gateway 10.10.10.1 | PASS |
| Internet connectivity | PASS |
| DNS resolution | PASS |
| SSH via 10.10.10.20 | PASS |
| Pi5 moved to permanent shelf | PASS |

## 10. Result

LAB-007 — Pi5 Network Migration: COMPLETE

The Raspberry Pi 5 was successfully migrated from the Airtel LAN management path to the AstraHM internal LAN.

The Pi5 is now reachable at:

10.10.10.20

The network path, gateway, Internet connectivity, DNS resolution, and remote SSH access were verified successfully.

## 11. Next Steps

- Verify permanent hostname configuration.
- Decide whether Wi-Fi should remain enabled as a recovery path.
- Configure SSH key authentication.
- Disable root/password SSH after key authentication is verified.
- Inventory and document NVMe devices.
- Prepare Pi5 for storage and Lustre engineering experiments.
