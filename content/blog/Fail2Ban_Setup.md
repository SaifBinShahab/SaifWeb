+++
title = "SSH Brute Force Protection with Fail2Ban"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "DevOps", "Security", "Server"]
+++
Fail2Ban is an Intrusion Prevention framework written in python which protects any protcol that has an "username" and "password" field. Like SSH, FTP, Telnet an so on.

# Some pre-checks (Optional)
## First check SSH config
```sh
sudo vim /etc/ssh/sshd_config
```

In the SSH config file the line `MaxAuthTries` is also respected by fail2ban. So, we should also provide the maximum number of failed atemts we want to have. We are going to set the same value in fail2ban config. In this case we are giving it a value of "3".
```sh
MaxAuthTries 3
```

### Check the `auth.log` file. Check for which filter to use.

fail2ban looks at the `/var/log/auth.log` file to monitor failed login attempts.
```sh
cat /var/log/auth.log | grep "sshd
```

In this case we can see grepping for "sshd" gives us the logs for ssh. So, "sshd" will be our filter

# Setup fail2ban

## Install fail2ban
```sh
sudo apt install -y fail2ban
```


## Configure fail2ban

### Create fail2ban jail
```sh
sudo vim /etc/fail2ban/jail.local
```

Put the followin config in the `jail.local` file.

```sh
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
findtime = 300
bantime = 3600
```

### Legend:
- `filter`: The string to look for in the log file to mintor.
- `logpath`: The log file to monitor.
- `maxretry`: Max number of retries or failed login attemts.
- `findtime`: Max time in seconds between each failed login attempts. If the time is less than this for the failed attempts then it will be considered as a bruteforce attempt.
- `bantime`: The time in seconds for how long the ip will be banned for retrying again.

Restart the service
```sh
sudo systemctl restart fail2ban.service
```


## Note
Fail2ban will respect the `MaxAuthTries` entry in `sshd_config` file. That's why it will priorities the `sshd_config` file before its own jail config. For example,
- Case 1: if `MaxAuthTries` is set to 6 and fail2ban jail config has `maxretry` set to 3 then fail2ban will block the ip after 6 auth tries (NOT retries). 
- Case 2: On the contrary, if the `sshd_config` file has `MaxAuthTries` set to 2 and fail2ban has `maxretry` set to 3 then fail2ban will block the ip after 2 auth tries (NOT retries).
- Case 3: If we want the ip to be blocked after 3 auth failures/tries then we need to set both `MaxAuthTries` in `sshd_config` and `maxretry` in fail2ban jail to 3.

## Unban banned IPs
You can check fail2ban status with:
```sh
fail2ban-client status sshd

# It will show banned IPs and stuff.
```

You can unban an IP with:
```sh
fail2ban-client set sshd unbanip 192.168.122.1
```

Done!
