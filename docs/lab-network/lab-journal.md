# AstraHM Lab Journal

## LAB-002 — LAN Foundation
**PASS**

- EdgeRouter LAN: 10.10.10.1/24
- DHCP: 10.10.10.100-199
- TP-Link: 10.10.10.2
- Admin account javed validated

## LAB-003 — Airtel WAN, NAT and DNS
**PASS**

WAN:
```
eth0 = 192.168.0.47/24
default route = 192.168.0.1
```

NAT:
```
10.10.10.0/24 -> eth0 masquerade
```

DNS:
```
listen-on switch0
upstream 192.168.0.1
```

Mac with Wi-Fi disabled successfully reached 8.8.8.8 and resolved google.com through 10.10.10.1.

## LAB-004 — Proxmox
**PASS**

Migrated from 192.168.0.2/24 to 10.10.10.10/24. Gateway 10.10.10.1. SSH/Web UI and Internet validated.

## LAB-005 — dev-box-01
**PASS**

Migrated to 10.10.10.30/24. Gateway/DNS 10.10.10.1. Internet/DNS validated.

## Baseline freeze

The network foundation is stable and should be committed before further network changes.