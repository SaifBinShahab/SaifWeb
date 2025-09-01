+++
title = "Install and Configure VNC server on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "Server"]
+++
## Step 1 — Installing the Desktop Environment and VNC Server

```sh
sudo apt update && sudo apt install -y xfce4 xfce4-goodies tightvncserver dbus-x11
```

To complete the VNC server’s initial configuration after installation, use the vncserver command to set up a secure password and create the initial configuration files:
```sh
vncserver
```

Next there will be a prompt to enter and verify a password to access your machine remotely:

Output:
```sh
You will require a password to access your desktops.

Password:
Verify:
```

The password must be between six and eight characters long. Passwords with more than eight characters will be truncated automatically.

Once you verify the password, you have the option to create a view-only password. Users who log in with the view-only password will not be able to control the VNC instance with their mouse or keyboard. This is a helpful option if you want to demonstrate something to other people using your VNC server, but this isn’t required.

The process then creates the necessary default configuration files and connection information for the server:

Output:
```sh
Would you like to enter a view-only password (y/n)? n
xauth:  file /home/sammy/.Xauthority does not exist

New 'X' desktop is your_hostname:1

Creating default startup script /home/sammy/.vnc/xstartup
Starting applications specified in /home/sammy/.vnc/xstartup
Log file is /home/sammy/.vnc/your_hostname:1.log
```
Next, configure it to launch Xfce and give access to the server through a graphical interface.

&nbsp;

## Step 2 — Configuring the VNC Server
The VNC server needs to know which commands to execute when it starts up. Specifically, VNC needs to know which graphical desktop it should connect to.

These commands are located in a configuration file called xstartup in the .vnc folder under your home directory. The startup script was created when you ran the vncserver command in the previous step, but you’ll create your own to launch the Xfce desktop.

When VNC is first set up, it launches a default server instance on port 5901. This port is called a display port, and is referred to by VNC as :1. VNC can launch multiple instances on other display ports, like :2, :3, and so on.

Because you are going to be changing how the VNC server is configured, first stop the VNC server instance that is running on port 5901 with the following command:
```sh
vncserver -kill :1
```

Output
```sh
Killing Xtightvnc process ID 17648
```

Before you modify the xstartup file, back up the original:
```sh
mv ~/.vnc/xstartup ~/.vnc/xstartup.bak
```

Now create a new xstartup file and open it in your preferred text editor:
```sh
vim ~/.vnc/xstartup
```

Commands in this file are executed automatically whenever you start or restart the VNC server. You need VNC to start your desktop environment if it’s not already started. Add the following commands to the file:
```sh
#!/bin/sh

xrdb "$HOME/.Xresources"
xsetroot -solid grey
#x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
#x-window-manager &
# Fix to make GNOME work
export XKL_XMODMAP_DISABLE=1
/etc/X11/Xsession
startxfce4 &
```

Here is a brief overview of what each line is doing:

- `#!/bin/bash`: The first line is a shebang. In executable plain-text files on *nix platforms, a shebang tells the system what interpreter to pass that file to for execution. In this case, you’re passing the file to the Bash interpreter. This will allow each successive line to be executed as commands, in order.
- `xrdb $HOME/.Xresources`: This command tells VNC’s GUI framework to read the user’s `.Xresources` file. `.Xresources` is where a user can make changes to certain settings for the graphical desktop, like terminal colors, cursor themes, and font rendering.
- `startxfce4 &`: This command tells the server to launch Xfce. This is where you will find all the graphical software that you need to comfortably manage your server.

To ensure that the VNC server will be able to use this new startup file properly, you need to make it executable:
```sh
sudo chmod +x ~/.vnc/xstartup
```

Now, restart the VNC server:
```sh
vncserver
```

The output will be similar to the following:
Output
```sh
New 'X' desktop is your_hostname:1

Starting applications specified in /home/sammy/.vnc/xstartup
Log file is /home/sammy/.vnc/your_hostname:1.log
```

With the configuration in place, you’re ready to connect to the VNC server from your local machine.

&nbsp;

## Step 3 - Make it persist across reboots and shutdowns

### Running VNC as a System Service
Next, you’ll set up the VNC server as a systemd service. You can start, stop, and restart it as needed, like any other service. This will also ensure that VNC starts up when your server reboots.

First, create a new unit file called `/etc/systemd/system/vncserver@.service` using your favorite text editor:
The @ symbol at the end of the name will let you pass in an argument you can use in the service configuration. You’ll use this to specify the VNC display port you want to use when you manage the service.
```sh
sudo vim /etc/systemd/system/vncserver@.service
```

Add the following lines to the file. Be sure to change the value of User, Group, WorkingDirectory, and the username in the value of PIDFILE to match your username:
```sh
[Unit]
Description=Start TightVNC server at startup
After=syslog.target network.target

[Service]
Type=forking
User=debian
Group=debian
WorkingDirectory=/home/debian

PIDFile=/home/sammy/.vnc/%H:%i.pid
ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1
ExecStart=/usr/bin/vncserver -depth 24 -geometry 1280x800 :%i
ExecStop=/usr/bin/vncserver -kill :%i

[Install]
WantedBy=multi-user.target
```

The `ExecStartPre` command stops VNC if it’s already running. The `ExecStart` command starts VNC and sets the color depth to 24-bit color with a resolution of 1280x800. You can modify these startup options as well to meet your needs.

Save and close the file when you’re finished.

Next, make the system aware of the new unit file:
```sh
sudo systemctl daemon-reload
```

Then, enable the unit file:
```sh
sudo systemctl enable vncserver@1.service
```

The 1 following the @ sign signifies which display number the service should appear over, in this case the default :1 as was discussed in Step 2.

Stop the current instance of the VNC server if it’s still running.
```sh
vncserver -kill :1
```

Then start it as you would start any other systemd service.
```sh
sudo systemctl start vncserver@1
```

You can verify that it started with the following command:
```sh
sudo systemctl status vncserver@1
```

If it started correctly, the output will be similar to the following:
Output
```
● vncserver@1.service - Start TightVNC server at startup
Loaded: loaded (/etc/systemd/system/vncserver@.service; enabled; vendor preset: enabled)
Active: active (running) since Fri 2022-08-19 16:21:36 UTC; 5s ago
Process: 24469 ExecStartPre=/usr/bin/vncserver -kill :1 > /dev/null 2>&1 (code=exited, status=2)
Process: 24474 ExecStart=/usr/bin/vncserver -depth 24 -geometry 1280x800 -localhost :1 (code=exited, status=0/SUCCESS)
Main PID: 24482 (Xtightvnc)
. . .
```



### Running VNC with cronjob
```sh
sudo crontab -e

# Add the following line and save

@reboot /usr/bin/vncserver
```


## On the client machine

Install "Remmina"
```sh
sudo pacman -S remmina
```

Add a new connection and in protocol select "Remmina VNC Plugin..." Then provide in the server field:  `IP:Port` of the server. In this case, `192.168.0.something:5901`

Done!
