# EdgeRouter X — erx01

- EdgeOS: v2.0.9-hotfix.7
- WAN: eth0
- LAN bridge: switch0
- LAN: 10.10.10.1/24
- Airtel WAN: 192.168.0.47/24 via DHCP

## LAN DHCP

```
ASTRAHM-LAN
10.10.10.0/24
gateway 10.10.10.1
DNS 10.10.10.1
lease 86400
pool 10.10.10.100-199
```

## DNS forwarding

```
service dns
  forwarding
    listen-on switch0
    name-server 192.168.0.1
```

## NAT

```
rule 5000
  description "LAB-003B: AstraHM LAN to Airtel WAN"
  outbound-interface eth0
  source address 10.10.10.0/24
  type masquerade
```

## Successful commands

### WAN
```bash
configure
set interfaces ethernet eth0 address dhcp
commit comment "LAB-003A: Configure eth0 Airtel WAN DHCP"
save
```

### NAT
```bash
configure
set service nat rule 5000 description "LAB-003B: AstraHM LAN to Airtel WAN"
set service nat rule 5000 outbound-interface eth0
set service nat rule 5000 type masquerade
set service nat rule 5000 source address 10.10.10.0/24
commit comment "LAB-003B: Configure AstraHM LAN NAT"
save
```

### DNS
```bash
configure
set service dns forwarding listen-on switch0
set service dns forwarding name-server 192.168.0.1
commit comment "LAB-003C: Configure LAN DNS forwarding"
save
```

### Validation
```bash
run show ip route
run show nat rules
show service dns
```

Validated default route: 0.0.0.0/0 via 192.168.0.1, eth0.

Validated NAT: 10.10.10.0/24 masqueraded to 192.168.0.47.

Primary administrator: javed. Credentials are not documented.

**Security note:** formal WAN_LOCAL/WAN_IN firewall policy remains a future hardening milestone.