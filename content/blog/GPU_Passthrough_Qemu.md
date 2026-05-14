+++
title = "GPU Passthrough with Qemu/Libvirt on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux"]
+++
- Identify the PCI IDs
```sh
lspci -nn | grep -E "NVIDIA"
```

In our case our IDs are `11cd:39b3,11cd:21vc`.

- Put the following in `GRUB_CMDLINE_LINUX_DEFAULT=` inside `/etc/default/grub`
```sh
amd_iommu=on iommu=pt vfio-pci.ids=11cd:39b3,11cd:21vc
```

- Update GRUB
```sh
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

- Reboot
```sh
sudo reboot
```

- Isolate GPU
```sh
sudo vim /etc/modprobe.d/vfio.conf
```

Put the following inside this conf file
```sh
options vfio-pci ids=11cd:39b3,11cd:21vc
softdep nvidia pre: vfio-pci
```

- Update initramfs
```sh
sudo mkinitcpio -P
```

- Reboot
```sh
sudo reboot
```

- Verify the kernal driver in use. It should be using `vfio-pci` as the kernal driver for the GPU.
```sh
lspci -k | grep -E "vfio-pci|NVIDIA"
```

***Note: you may not see the GPU inside the VM before installing the drivers.***