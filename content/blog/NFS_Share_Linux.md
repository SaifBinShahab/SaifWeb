+++
title = "Setting up NFS Share on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "Server"]
+++
# On_Server
- Install required server components.
```sh
sudo apt install nfs-kernel-server
```

- Check if the service is alive.
```sh
sudo systemctl status nfs-kernel-server.service
```

- Create the export directory for the NFS share.
```sh
sudo mkdir -p /nfs/share/project1
```

- Edit the /etc/exports file.
A client can be identified either through a specific IP address, an entire subnet, or a domain name.
```sh
sudo vim /etc/exports
--------------------------------
# Usage
export_directory_name client1(sharing_options) client2(sharing_options_2)

# Example
/nfs/share/project1  10.10.11.213(rw,sync,no_subtree_check)
```

- Apply the configuration changes using the `exportfs` command.
```sh
sudo exportfs -a
```

- View all the active exports and verify the `/nfs/share/project1` directory has export status.
```sh
sudo exportfs -v
------------------------
/nfs/share/project1 10.10.11.213(rw,wdelay,root_squash,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```

- Restart the NFS utility using `systemctl`.
```sh
sudo systemctl restart nfs-kernel-server
```

# On_Client
- Create the directory where you wanna mount the share.
```sh
sudo mkdir -p /nfs/mnt/project1
```

- Mount the share.
```sh
# Usage
sudo mount -t nfs  server_ip_addr:/nfs/share/project1  /nfs/mnt/project1

# Example
sudo mount MyCutieServer:/Data/share /sharedata
```

## Persistent mount (Client)
The mount only persists until the system reboots. To automatically mount the directory when the system activates, add the mount point to the `/etc/fstab` file. The entry should consist of the export directory, the local mount point, a list of options, and two `0`’s. To see all the available options for this file, run `man nfs` on the client.
```sh
sudo vim /etc/fstab
----------------------------
server_ip_addr:/nfs/share/project1  /nfs/mnt/project1 nfs timeo=900,intr,actimeo=1800 0 0
```

# Configuring the Firewall (Server)
Add a rule to allow port `2049` to accept traffic from the client’s IP address. Replace `client_ip_addr` with the address of the client.
````sh
sudo ufw allow from client_ip_addr to any port nfs
````

# Extras (Server)
- For security reasons, NFS translates any root credentials from a client to nobody:nogroup. This restricts the ability of remote root users to invoke root privileges on the server. To grant client root users access to the export directory, change the directory ownership to nobody:nogroup.

```sh
sudo chown nobody:nogroup /nfs/share/project1
```
