+++
title = "Setting up Wake On LAN on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "Server"]
+++
## On the machine to wake

Install the "ethtool" package.
```sh
sudo apt install -y ethtool
```

Check the network interface MAC address.
```sh
ip a
```

Check the wake-on-lan state:
```sh
sudo ethtool eth0 | grep Wake-on
```

Sample Output:
```sh
debian@debian:~$ sudo ethtool eth0 | grep Wake-on
[sudo] password for debian: 
	Supports Wake-on: pumbg
	Wake-on: d
debian@debian:~$ 
```

The "d" state means "disabled". So, enable it.
```sh
sudo ethtool -s eth0 wol g
```

Now, it's enabled. Now we gotta make it persist throghout reboots and shutdowns.
```sh
sudo crontab -e

# Add the following line at the end and save it.

@reboot /usr/sbin/ethtool -s eth0 wol g
```

&nbsp;

## On the main machine from where it will send the wake-on-lan signal aka magick packet

Install the "wakeonlan" package.
```sh
sudo pacman -S wakeonlan
```

You're done!
```sh
Syntax:
wakeonlan [MAC_OF_SERVER]

Example:
wakeonlan 76:f9:27:1c:65:43
```
