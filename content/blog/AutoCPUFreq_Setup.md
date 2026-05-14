+++
title = "Installing Auto CPU Freq on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "Server"]
+++
```sh
sudo apt install -y acpi acpi-support acpitool battery-stats
```
```
git clone https://github.com/AdnanHodzic/auto-cpufreq.git
cd auto-cpufreq && sudo ./auto-cpufreq-installer
```

Install the daemon using CLI (after installing auto-cpufreq):

Installing the auto-cpufreq daemon using CLI is as simple as running the following command:

```
sudo auto-cpufreq --install
```
After the daemon is installed, auto-cpufreq is available as a binary and runs in the background. Its stats can be viewed by running: `auto-cpufreq --stats`
