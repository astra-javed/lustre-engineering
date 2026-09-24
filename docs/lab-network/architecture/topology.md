# AstraHM Network Topology

## Physical topology

```
Airtel LAN 192.168.0.1
        |
        v
EdgeRouter X erx01
        |
      eth1
        |
        v
TP-Link TL-SG108E sw01
        |
   +----+----+----------------+
   |         |                |
Proxmox   dev-box-01         Mac
10.10.10.10 10.10.10.30      en8
```

## Logical topology

```
192.168.0.0/24
Airtel upstream
       |
EdgeRouter eth0
192.168.0.47 DHCP
       |
NAT + DNS forwarding
       |
switch0 10.10.10.1/24
       |
10.10.10.0/24 AstraHM Lab LAN
```

VLANs are intentionally deferred until this flat-LAN baseline is frozen.