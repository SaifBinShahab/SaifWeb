+++
title = "Encrypting Disk on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux"]
+++
## Make sure the encryption modules are loaded
```sh
sudo modprobe dm-crypt
sudo modprobe dm-mod
```
&ensp;

## Prepare the disk
 - Wipe the disk, delete all the partition. Basically we're gonna need a clean disk.
 - You can securely erase the disk with various methods. Which basically overwrites the disk with random data that prevents potential data recovery attempts. You need to do this only once in the lifetime for a disk.
 - ***It is not recommended to overwrite an SSD if you plan on using `fstrim`.***
&ensp;

## Setup LUKS on the disk
```sh
sudo cryptsetup luksFormat -v -s 512 -h sha512 /dev/nvme1n1
```

- **Note:** The device can either be a disk itself or a partition.
- Then type "YES" for the confirmation.
- Then enter the passphrase. You'll need to entire it twice.

## Open the encrypted partition
```sh
# Syntax
sudo cryptsetup open /dev/nvme1n1 --type luks <cryptdiskname>

# Example
sudo cryptsetup open /dev/nvme1n1 --type luks cryptdisk
```

- After unlocking the partition, it will be available at `/dev/mapper/cryptdisk`. Now create a file system of your choice.
&ensp;

## Partition the disk
In this example we're gonna use `btrfs`.
```sh
# Syntax
mkfs.fstype -L mylabel /dev/mapper/name

# Example
mkfs.btrfs -L Disk /dev/mapper/cryptdisk
```

- So, basically we **setup encryption** on a clean disk, then **open** the encrypted disk, then we **create partition** on that decrypted device which is a mapped device. (clean disk > encryption > partition) -- Used in This example.
- Or, we can create a partition on a clean disk and add encryption on that partition. (clean disk > partition > encryption)

## Manual mounting and unmounting

- To mount the partition:
```sh
sudo cryptsetup open --type luks cryptdisk

# Syntax
mount -t fstype /dev/mapper/cryptdisk /mnt/disk

# Example
mount -t btrfs /dev/mapper/cryptdisk /mnt/disk
```

- To unmount it:
```sh
umount /mnt/disk

cryptsetup close cryptdisk
```

- Mounting with a file manager (i.e. Dolphin, Thunar) requires `gvfs` installed.
&ensp;
