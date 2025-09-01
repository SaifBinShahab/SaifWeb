+++
title = "Getting Nvidia GPU working on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux"]
+++
## \# Setting up

### \# Nvidia propieretory drivers and stuff
```sh
sudo pacman -S nvidia-dkms opencl-nvidia lib32-opencl-nvidia libglvnd nvidia-utils lib32-libglvnd lib32-nvidia-utils nvidia-settings

# Note: If using "linux" kernal install "nvidia" driver. For "linux-zen" install "nvidia-dkms".
# Also remove the package "optimus-manager" because it blacklists nvidia kernel modules.
```

&nbsp;

### \# Grub config
Add `nvidia-drm.modeset=1 rd.driver.blacklist=nouveau modprobe.blacklist=nouveau` in `GRUB_CMDLINE_LINUX_DEFAULT=` inside the file `/etc/default/grub`

```sh
sudo vim /etc/default/grub
```

Put the cmdline parameters
```sh
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet nvidia-drm.modeset=1 rd.driver.blacklist=nouveau modprobe.blacklist=nouveau"
```

&nbsp;

Then generate grub config
```sh
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

&nbsp;

### \# Initramfs
```sh
sudo vim /etc/mkinitcpio.conf
```

```sh
MODULES=(amdgpu nvidia nvidia_modeset nvidia_uvm nvidia_drm)
```

```sh
sudo mkinitcpio -P
```

&nbsp;

### \# Now Reboot. 
`sudo reboot now`

&nbsp;

### \# Check if the modules are loaded successfully
```sh
lsmod | grep nvidia
```

This should output something like this:
```sh
➜ ~ lsmod | grep nvidia
nvidia_drm            122880  5
nvidia_uvm           6602752  0
nvidia_modeset       1605632  3 nvidia_drm
video                  77824  3 amdgpu,ideapad_laptop,nvidia_modeset
nvidia              60506112  29 nvidia_uvm,nvidia_modeset
```

&nbsp;

## \# Run Aplications using nvidia gpu

### \# Prepend these Environment Variables before applications to run with nvidia gpu:
`DRI_PRIME=pci-0000_01_00_0 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia`

```sh
Syntax:
DRI_PRIME=pci-0000_01_00_0 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia <command>

Example:
DRI_PRIME=pci-0000_01_00_0 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia shotcut
DRI_PRIME=pci-0000_01_00_0 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia glxinfo -B
DRI_PRIME=pci-0000_01_00_0 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia vulkaninfo
```

&nbsp;

### \# Or, just use `prime-run` command
```sh
Syntax:
prime-run <command>

Example:
prime-run shotcut
prime-run glxinfo -B
prime-run vulkaninfo
```

&nbsp;

### \# You can check which programs are using nvidia gpu
```sh
nvidia-smi
```

&nbsp;

## Troubleshooting
If you run into problem with CUDA then try running `nvidia-modprobe`.

&nbsp;

## Done!
