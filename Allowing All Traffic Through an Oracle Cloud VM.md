---
title: Allowing All Traffic Through an Oracle Cloud VM
created: '2021-05-11T19:33:30.537Z'
modified: '2021-05-12T10:53:49.632Z'
---

# Allowing All Traffic Through an Oracle Cloud VM
## Using IPTABLES to allow all traffic
To see current IPTABLES rules:
```sh
sudo iptables -L
```

To save current IPTABLES rules to a file:
```sh
sudo iptables-save > ~/iptables-rules
```

Effectively *DISABLE* IPTABLES by following commands:
```sh
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -F
```

## Saving IPTABLES Configuration
The IPTABLES configuration is not persistent and will be reset at next boot, to make it persistent, you'll have to install *iptables-persistent*
```sh
sudo apt-get install iptables-persistent
```

To save current IPTABLES configuration:
```sh
sudo netfilter-persistent save
```
