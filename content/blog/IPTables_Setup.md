+++
title = "Setting up Firewall with IPTables"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "DevOps", "Security", "Server"]
+++
- Flush all `iptables` rules.
```sh
sudo iptables -F
```

- List all rules.
```sh
sudo iptables -L

# With line numbers
sudo iptables -L --line-numbers
```

- Change default `INPUT` policy from `ACCEPT` to `DROP`.
  ***WARNING!: MIGHT BLOCK YOUR SSH CONNECTIONS!***
  - `-P` or `--policy` to set default policy.
```sh
sudo iptables --policy INPUT DROP
```

- Block All connections from an IP.
  - `-I` for Inserting the rule. `-A` to Append.
  - `-s` is Source IP.
  - `-j` is the Operation/Target (`ACCEPT`/`DROP`/`REJECT`).
```sh
# Block only one IP
sudo iptables -I INPUT -s 10.0.0.11 -j DROP

# Block an entire subnet
sudo iptables -I INPUT -s 10.0.0.11/24 -j DROP
```

- Delete rules.
First list the rules with line numbers to see the line number of the rule you wanna delete.
```sh
# Usage
sudo iptables -D <Chain> <Line_Number>

# Example
sudo iptables -D INPUT 1
```

- Block all connections to a specific port on the server.
```sh
sudo iptables -I INPUT -p tcp --dport 80 -j DROP
```

- Accept from an IP to a port on the server.
```sh
sudo iptables -I INPUT -p tcp --dport 80 -s 10.10.11.213 -j ACCEPT
```

## Making Changes permanent

As IP-Tables are not persistent, they will be deleted ("flushed") with the next reboot.

- Once you are happy with your ruleset, save the new rules to the master iptables file:
```sh
iptables-save > /etc/iptables.up.rules
```

- To make sure the iptables rules are started on a reboot we'll create a new file:
```sh
vim /etc/network/if-pre-up.d/iptables
```
- Add these lines to it:
```sh
 #!/bin/sh
 /sbin/iptables-restore < /etc/iptables.up.rules
```

- The file needs to be executable so change the permissions:
```sh
chmod +x /etc/network/if-pre-up.d/iptables
```

- Rules can be stored something like this:
```sh
iptables-save > /etc/iptables/rules.v4
ip6tables-save > /etc/iptables/rules.v6
```

### Another way is to use the package `iptables-persistent`.
- Install it.
```sh
sudo apt install iptables-persistent
```

- Enable the service.
```sh
sudo systemctl enable netfilter-persistent.service
sudo systemctl status netfilter-persistent.service
```

# Useful Rules
- Accepts all established inbound connections (IMPORTANT!)
```sh
sudo iptables -I INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

 - Allow SSH, HTTP, HTTPS.
```sh
sudo iptables -A INPUT -p tcp -s src_ip --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

- Allow ICMP ping (echo).
>Note that blocking other types of icmp packets is considered a bad idea by some
>remove -m icmp --icmp-type 8 from this line to allow all kinds of icmp:
>https://security.stackexchange.com/questions/22711

```sh
sudo iptables -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
```

- Default Deny Input and Allow Output.
```sh
sudo iptables -A OUTPUT -j ACCEPT
sudo iptables -A INPUT -j DROP
sudo iptables -A FORWARD -j DROP
```


Referances:
- [https://wiki.debian.org/iptables](https://wiki.debian.org/iptables)
- [https://www.cyberciti.biz/faq/how-to-save-iptables-firewall-rules-permanently-on-linux/](https://www.cyberciti.biz/faq/how-to-save-iptables-firewall-rules-permanently-on-linux/)
