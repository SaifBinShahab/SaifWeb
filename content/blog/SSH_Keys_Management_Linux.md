+++
title = "SSH Keys Management on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "DevOps", "Homelab", "Server"]
+++
## Generate SSH Keys

- With ED25519 (Recommended)
```
ssh-keygen -t ed25519 -C "johndoe@email.com"
```
&ensp;

- With RSA
```
ssh-keygen -t rsa -b 4096 -C "johndoe@email.com"
```
&ensp;

## Multiple SSH Keys Management
- Generate multiple SSH Keys for your multiple servers.
- When generating the keys use different filenames for different servers.
- Put the configs in the `~/.ssh/config` file:

```
# Server 1
Host MyServer # Give a short name
HostName blog.somewebsite.com # Domain or IP
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_somewebsite # The Private Key
User ubuntu # The username

# Server 2
Host c2 # Give a short name
HostName 172.117.93.121 # Domain or IP
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_ed25519_secretserver # The Private Key
User root # The username
```
&ensp;

- When SSHing into a server simply provide the `Host` like this:

```
ssh MyServer
```
&ensp;

***Note: Now, when you are cloning the **repository** (**named demo**) from the company's GitHub account, the repository URL will be something like:***

`Repo URL: git@github.com:abc1234/demo.git`

Now, while doing `git clone`, you should modify the above repository URL as:

`git@company:abc1234/demo.git`

**Notice how github.com is now replaced with the alias "company" as we have defined in the configuration file.**

Similary, you have to modify the clone URL of the repository in the personal account depending upon the alias provided in the configuration file.
&ensp;

## Transfer the public key securely
```
ssh-copy-id -i ~/.ssh/id_ed25519_secretserver.pub root@172.117.93.121
```
&ensp;
