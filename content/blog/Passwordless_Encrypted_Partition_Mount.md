+++
title = "How to Mount an Encrypted LUKS Partition without Password using Keyfile"
date = 2025-08-25
[taxonomies]
tags = ["Linux"]
+++
## Setup the keyfile

- Create the keyfile with which we're gonna decrypt the encrypted partition.
```sh
dd if=/dev/random of=/secure/keyfile.bin bs=512 count=8
```

- The default location for cryptsetup key files is `/etc/cryptsetup-keys.d/cryptdevicename.key`. If no kefile is mention in the `crypttab` file then systemd will look into that default location.

- Add the key to the disk header.
```sh
sudo cryptsetup -v luksAddKey /dev/sda4 /secure/keyfile.bin
```

- Check if the keyfile was added successfully. If you see another key slot is enabled (In this case the second key or Key slot 1), then it was successful.
```sh
$ sudo cryptsetup luksDump /dev/sdb1 | grep "Key Slot"
Key Slot 0: ENABLED
Key Slot 1: ENABLED
Key Slot 2: DISABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```

Or,

```sh
$ sudo cryptsetup luksDump /dev/sdb1 | grep "KeySlots"
```

- Check if the encrypted partition unlockes with that keyfile.
```sh
$ sudo cryptsetup -v luksOpen /dev/sdb1 sdb1_crypt --key-file=/etc/luks-keys/disk_secret_key
Key slot 1 unlocked.
Command successful.
```

- Close the encrypted partition.
```sh
$ sudo cryptsetup -v luksClose sdb1_crypt
Command successful.
```
&ensp;

## Auto mount on boot time
- Get the `UUID` of the encrypted partition.
```sh
$ sudo cryptsetup luksDump /dev/sdb1 | grep "UUID"
UUID:          	2a2375bf-2262-413c-a6a8-fbeb14659c85
```

- The `/etc/crypttab` config file to decrypt and mount the encrypted partition on `/dev/mapper/`.
```sh
# /etc/crypttab configuration file 

Data UUID=2a2375bf-2262-413c-a6a8-fbeb14659c85 /secure/keyfile.bin luks,nofail
```
The `nofail` option is to be used for secondary non-root partitions on which the boot does not rely. The `nofail` option allows the system to continue boot without waiting for this partition to decrypt. Otherwise, the system will wait for it to decryp which will slow down boot time.

- On `systemd` systems perform:

```sh
sudo systemctl daemon-reload
```

- Check if the config is working.
```sh
$ sudo cryptdisks_start sdb1_crypt
 * Starting crypto disk...
 * sdb1_crypt (starting)..
 * sdb1_crypt (started)...                 [ OK ] 
```

Or,

```sh
$ sudo systemctl start systemd-cryptsetup@cryptssd.service
$ sudo cryptsetup status cryptssd
/dev/mapper/cryptssd is active.
  type:    LUKS2
......
......
```

- Then configure the `/etc/fstab` config file to mount the mapped device.
```sh
# /etc/fstab file

/dev/mapper/Data /media/Data ext4    defaults   0       0
```

- It's better to limit the permission on the keyfile.
```sh
sudo chmod 000 /secure/keyfile.bin
```
&ensp;

Done!
