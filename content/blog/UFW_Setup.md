+++
title = "Setting up firewall with UFW on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "DevOps", "Security", "Server"]
+++
## Check UFW status
```sh
sudo ufw status
```

It may return status active or inactive depending on the default config. If it is already active then disable it first.

Btw, the default ufw config is located in `/etc/default/ufw`.

## Disable UFW
```sh
sudo ufw disable

sudo systemctl stop ufw
```

It will disable the firewall for now. Now if the firewall was active and it had some preconfigured rules, we need to reset everything to start with a clean environment.

## Reset the firewall rules (PROCEED WITH CAUTION!)
```sh
sudo ufw reset
```

## Configure UFW

### Default incoming and outgoing
```sh
sudo ufw default deny incoming

sudo ufw default allow outgoing
```

### Allow SSH connection
We can use both the name "ssh" or the port number "22"

```sh
sudo ufw allow ssh
```

### Allow HTTP and HTTPS connections
```sh
sudo ufw allow http
sudo ufw allow https
```

## Some Tests or Pratices with defferent scenarios (Optional)
### We can Allow/Deny FTP
```sh
sudo ufw allow ftp
sudo ufw deny ftp
```

### Allow/Deny traffic only from a specific IP address
```sh
sudo ufw allow from 10.0.0.1
sudo ufw deny from 10.0.0.1
```

### Allow SSH connection only from a specific IP
```sh
sudo ufw allow from 10.0.0.1 to any port 22
```

We can also specify an entire subnet like:
```sh
sudo ufw allow from 10.0.0.1/24 to any port 22
```

## Enable UFW
After configuring UFW we need to enable it to take efffect.

```sh
sudo systemctl start ufw

sudo ufw enable
```

## Managing UFW rules

### Print the status or rules in numbered format
```sh
sudo ufw status numbered
```

It shows an output similar to the following:
```sh
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 80/tcp                     ALLOW IN    Anywhere
[ 3] 443                        ALLOW IN    Anywhere
[ 4] 21/tcp                     ALLOW IN    Anywhere
[ 5] 22                         ALLOW IN    10.0.0.0/24
[ 6] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 7] 80/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 8] 443 (v6)                   ALLOW IN    Anywhere (v6)
[ 9] 21/tcp (v6)                ALLOW IN    Anywhere (v6)

```

### Delete a rule
First we need to print the status in numbered format. And we need to do that everytime we delete a rule. Because when we change the rules the index number changes.
For example, if we wanto delete rule number 5. To do that we can,
```sh
sudo ufw delete 5
```

Done!
