+++
title = "How to disable auto suspend when laptop lid closed on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux"]
+++

When a laptop's lid is closed the laptop goes to sleep/hibernate/suspend. This creates a problem if we are using the laptop as a server. To get rid of this issue we're going to tell the laptop to ignore the lid close event.

To do that,
```sh
sudo vim /etc/systemd/logind.conf

# And add these lines

HandleSuspendKey=ignore
HandleLidSwitch=ignore
HandleLidSwitchDocked=ignore
```

Done!
