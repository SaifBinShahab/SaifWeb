+++
title = "Setting up Jellyfin Server on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "Server"]
+++
```sh
sudo apt install -y curl gnupg
```
You can verify the script download integrity with (requires sha256sum):
```sh
diff <( curl -s https://repo.jellyfin.org/install-debuntu.sh -o install-debuntu.sh; sha256sum install-debuntu.sh ) <( curl -s https://repo.jellyfin.org/install-debuntu.sh.sha256sum )
```
An empty output means everything is correct. Then you can inspect the script to see what it does (optional but recommended) and execute it with:
```sh
less install-debuntu.sh
sudo bash install-debuntu.sh
```

If you run into any dependency errors, run this and it will install them and `jellyfin-ffmpeg`.

```sh
sudo apt install -f
sudo apt install -y ffmpeg jellyfin-ffmpeg # jellyfin-ffmpeg might not be available and youi might have to install jellyfin-ffmpeg6 or jellyfin-ffmpeg5. I installed jellyfin-ffmpeg5.
```

# HW acceleration

```sh
sudo apt install -y intel-opencl-icd intel-media-va-driver i965-va-driver mesa-opencl-icd
```

# jellyfin desktop client

```sh
yay -S jellyfin-media-player
```
