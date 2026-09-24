# Network Validation Checklist

## EdgeRouter
```bash
show interfaces ethernet eth0
run show ip route
run show nat rules
show service dns
```

## Mac with Wi-Fi disabled
```bash
route -n get 8.8.8.8
ping -c 4 10.10.10.1
ping -c 4 8.8.8.8
nslookup google.com
```

Expected Internet route:
```
gateway: 10.10.10.1
interface: en8
```

## Proxmox
```bash
ip addr show vmbr0
ip route
ping -c 4 10.10.10.1
ping -c 4 8.8.8.8
```

## dev-box-01
```bash
ip addr show ens18
ip route
cat /etc/resolv.conf
ping -c 4 10.10.10.1
ping -c 4 google.com
```

All baseline tests passed on 2026-09-23.