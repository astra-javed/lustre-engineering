# AstraHM Home Lab — Network Baseline

**Status: PASS — validated 2026-09-23**

This document set records the working AstraHM home-lab network.

## Topology

```
                         INTERNET
                            |
                     Airtel Router
                      192.168.0.1
                            |
                            | DHCP WAN
                            v
                    +----------------+
                    | EdgeRouter X   |
                    | erx01          |
                    | eth0 192.168.0.47
                    | switch0 10.10.10.1/24
                    | NAT + DNS      |
                    +-------+--------+
                            |
                            | eth1
                            v
                    +----------------+
                    | TP-Link sw01   |
                    | 10.10.10.2     |
                    +-------+--------+
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
          Proxmox       dev-box-01       Mac
        10.10.10.10    10.10.10.30      en8
```

## IP plan

| Device | Address | Role |
|---|---|---|
| Airtel router | 192.168.0.1 | Upstream gateway |
| EdgeRouter WAN | 192.168.0.47 DHCP | WAN |
| EdgeRouter LAN | 10.10.10.1/24 | Lab gateway/DNS |
| TP-Link sw01 | 10.10.10.2/24 | L2 management |
| Proxmox pve | 10.10.10.10/24 | Virtualization |
| Pi5 | 10.10.10.20 planned | Future node |
| Pi4 | 10.10.10.21 planned | Future node |
| dev-box-01 | 10.10.10.30/24 | Lustre development |
| MDS01 | 10.10.10.40 planned | Future Lustre MDS |
| OSS01 | 10.10.10.50 planned | Future Lustre OSS |
| OSS02 | 10.10.10.51 planned | Future Lustre OSS |
| Lustre client | 10.10.10.60 planned | Future client |
| DHCP pool | 10.10.10.100–199 | Dynamic clients |

## Completed milestones

- LAB-002 — EdgeRouter + TP-Link LAN: PASS
- LAB-003 — Airtel WAN + NAT + DNS: PASS
- LAB-004 — Proxmox migration: PASS
- LAB-005 — dev-box-01 migration: PASS

Freeze this baseline before further network changes.