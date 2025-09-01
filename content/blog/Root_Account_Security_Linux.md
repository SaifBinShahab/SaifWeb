+++
title = "Securing the root account on a Linux server"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "DevOps", "Security", "Server"]
+++
## Give the user administrative privileges
```sh
# Add the user "dev" to the sudo group
usermod -aG sudo dev
```

## Disable the Root account
***WARNING! PROCEED WITH CAUTION! MAKE SURE YOU HAVE ANOTHER USER SETUP AS A SUDOER BEFORE PROCEEDING. OTHERWISE, YOU MAY LOOSE ADMINISTRATIVE ACCESS!***

### Lockdown the Root account (Not Recommended)
**Prevents local login.**

***This method locks the root account and prevents user switching to the root account locally. Basically, it prevents password based logging in. Only way someone can access the rooot account is via valid SSH keys that has been setup previously.***

```sh
# Lock the root account
sudo passwd -l root

# Unlock it
sudo passwd -u root
```

### By disabling the Shell
**Prevents local and remote login.**

```sh
# Change the shell to "/usr/sbin/nologin"
sudo chsh root
# provide "/usr/sbin/nologin"
Changing the login shell for root
Enter the new value, or press ENTER for the default
        Login Shell [/bin/bash]: /usr/sbin/nologin

# To undo the changes and re-enable the root account. Change the shell manually in the "/etc/passwd" file.

sudo vim /etc/passwd

# The line should look lilke this
root:x:0:0:root:/root:/bin/bash
```
