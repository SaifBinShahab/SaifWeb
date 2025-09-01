+++
title = "SSH Security"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "DevOps", "Security", "Server"]
+++
## Create another user
```sh
useradd -m -s /bin/bash dev

passwd # Set a strong password

# switch to dev user
su dev

# switch to root user
su root - # the "-" is good to include to initialize the shell properly. it is optional though.
```

## Disable remote root login
```sh
sudo vim /etc/ssh/sshd_config

# find the line that says "PermitRootLogin" and set it to no. And save it.
PermitRootLogin no

# Restart ssh service to reload the config
sudo systemctl restart ssh
```

## Securing using SSH Keys

```sh
# on the client machine

# using a newer algorithm
ssh-keygen -t ed25519

# using rsa
ssh-keygen -t rsa -b 4096

# transfer the public key to the server securely
ssh-copy-id dev@192.168.122.246

# or, if you've provided custom keypair name, you have to provide the entire path of the key file.
ssh-copy-id -i /home/john/.ssh/ubnt_server.pub dev@192.168.122.246
```


## Disable Password Authentication
***`WARNING! BEFORE PROCEEDING MAKE SURE YOUR SSH KEYS BASED AUTHENTICATION IS WORKING PROPERLY. OTHERWISE DO NOT PROCEED. YOU MAY LOOSE ACCESS TO THE SERVER!`***

```sh
sudo vim /etc/ssh/sshd_config

# Find the line "PasswordAuthentication" and set it to "no". And save it
PasswordAuthentication no
```
