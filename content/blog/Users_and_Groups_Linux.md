+++
title = "Users and Groups Management in Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "DevOps", "Homelab", "Server"]
+++
## Add a user to a group
```sh
# "sudo" group in this example and user "dev"
Syntax:
usermod -aG <group> <user>

Example:
sudo usermod -aG sudo dev
```

## Add a user to multiple groups
```sh
Syntax:
usermod -aG <groups_seperated_with_commas> <user>

Example:
sudo usermod -aG sudo,network,wheel,power,kvm dev
```

## Remove a user from a group
```sh
# "sudo" group in this example and user "dev"
Syntax:
gpasswd -d <user> <group>

Example:
sudo gpasswd -d dev sudo
```
