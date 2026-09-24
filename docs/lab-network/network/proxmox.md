# Proxmox Network Baseline

- Hostname: pve
- Management: 10.10.10.10/24
- Gateway: 10.10.10.1
- Bridge: vmbr0
- Physical NIC: nic0

Target bridge configuration:

```
auto vmbr0
iface vmbr0 inet static
    address 10.10.10.10/24
    gateway 10.10.10.1
    bridge-ports nic0
    bridge-stp off
    bridge-fd 0
```

Validated:
```bash
ip addr show vmbr0
ip route
ping -c 4 10.10.10.1
ping -c 4 8.8.8.8
```

SSH and Web UI were successfully restored after migration from 192.168.0.2/24.