+++
title = "How to automatically suspend Linux server"
date = 2025-08-25
[taxonomies]
tags = ["Linux"]
+++
## Install Autosuspend
```sh
sudo apt install -y autosuspend
```
&ensp;

## The Plan
What we want is: The server will run from `0600 Hrs` to `1900 Hrs`. Meaning the server will wake up at `0600 Hrs` and go back to sleep (suspend) at `1900 Hrs`. But the server will only go to sleep only if all conditions fill up. For this there are "`Activity Checks`". We want the server to stay ALIVE when we are doign some work with the server or basically the server is in some use. Like, we are SSH'ed into the server, we are watching Jellyfin, using Nextcloud transfering files etc. These will be done through some "`Activity Checks`". Like `Active Connections, Network Load, System Load` etc. The Primary time (`0600 Hrs to 1900 Hrs`) will be given to `autosuspend` in the form of a `iCalendar` file (`.ics`). And the other checks will make sure that even if the time to sleep comes the server will not sleep untill we are done with the server. And we can always wake up the system before the given time with "`Wake On Lan`".
&ensp;

## Create `.ics` file
Install Mozilla Thunderbird on your computer and create an event. Then in that event the start time should be the desired start time or wake time of the server and the end time should be the desired sleep time of the server. Then click on that event and Copy it (`CTRL+c`) and paste it is a new plain text file and save it as a `.ics` file.

Or, you can simply edit the following `.ics` file and change the `DTSTART` and `DTEND` times as required. In this example the event is created such that the server will start at `0600 Hrs` and sleep at `1900 Hrs`.

```
BEGIN:VCALENDAR
PRODID:-//Mozilla.org/NONSGML Mozilla Calendar V1.1//EN
VERSION:2.0
BEGIN:VEVENT
CREATED:20240908T095140Z
LAST-MODIFIED:20240908T095222Z
DTSTAMP:20240908T095222Z
UID:aa12bb1b-8ce7-43f1-8c71-74771f7a23a0
SUMMARY:System_Up
RRULE:FREQ=DAILY
DTSTART;TZID=Asia/Dhaka:20240908T060000
DTEND;TZID=Asia/Dhaka:20240908T190000
TRANSP:OPAQUE
END:VEVENT
END:VCALENDAR

```
&ensp;

In this file, the value of `DTSTART` and `DTEND` represents the time in ISO-8601 format which is basically `YYYY-MM-DDTHH:MM:SS`. In this case it doesn't have the semicolons in between them. The `T` in the value is the seperator for the time, which means after the `T` the time is specified and before that is the date. The time here is in the following format, `HHMMSS`.
&ensp;

## The Configuration

Now we will backup the existing default configuration which resides in `/etc/autosuspend.conf`.
```sh
sudo cp /etc/autosuspend.conf /etc/autosuspend.conf.backup
```
&ensp;

Then we will create our own config file with the same name.
```sh
sudo vim /etc/autosuspend.conf
```
&ensp;

Then put the following config which has the desired config that we need. The config is pretty self explainatory.
```sh
# General configuration
[general]
interval = 30
idle_time = 900
suspend_cmd = /bin/systemctl suspend
wakeup_cmd = sh -c 'echo 0 > /sys/class/rtc/rtc0/wakealarm && echo {timestamp:.0f} > /sys/class/rtc/rtc0/wakealarm'
woke_up_file = /var/run/autosuspend-just-woke-up
lock_file = /var/lock/autosuspend.lock
lock_timeout = 30

# Activity Checks
[check.RemoteUsers]
class = Users
enabled = true
name = .*
terminal = .*
host = [0-9].*

# Here the Users activity check is used again with different settings and a different name
[check.LocalUsers]
class = Users
enabled = true
name = .*
terminal = .*
host = localhost

[check.ActiveCalendarEvent]
enabled = true
url = file:///etc/System_up.ics

[check.ActiveConnection]
enabled=true
ports=22,80,9999

[check.Load]
enabled=true

[check.NetworkBandwidth]
enabled=true
interfaces=eth0

# Checks to determine the next scheduled wakeup are prefixed with 'wakeup'.
[wakeup.Calendar]
enabled = true
url = file:///etc/System_up.ics

# Apart from this, wake up checks reuse the same configuration mechanism.

```
&ensp;

## Enable and Start the Autosuspend daemon
```sh
sudo systemctl enable autosuspend.service
sudo systemctl enable autosuspend-detect-suspend.service

# Note: Do not forget the second enable call to ensure that wake ups are configured even if the system is manually placed into suspend.

sudo systemctl start autosuspend.service
```

Done!
