# dev-box-01 Network Baseline

- FQDN: dev-box-01.lab.astrahm.com
- Interface/profile: ens18
- IP: 10.10.10.30/24
- Gateway: 10.10.10.1
- DNS: 10.10.10.1
- Search domain: lab.astrahm.com

## NetworkManager

```bash
nmcli connection modify ens18 \
  ipv4.method manual \
  ipv4.addresses 10.10.10.30/24 \
  ipv4.gateway 10.10.10.1 \
  ipv4.dns 10.10.10.1
nmcli connection down ens18
nmcli connection up ens18
```

Validated:
```
default via 10.10.10.1 dev ens18
10.10.10.0/24 dev ens18 src 10.10.10.30
nameserver 10.10.10.1
search lab.astrahm.com
```

LAN and Internet/DNS tests passed.

## Development workflow

Access remotely with SSH and keep work in tmux:
```bash
ssh javed@10.10.10.30
tmux ls
tmux attach -t 0
```

Do not store passwords or private keys in documentation.