# ubuntu_chrome_remote_desktop
## How to get barrier and chrome remote desktop working in ubuntu 20.04

This is a mostly step-by-step guide on how to get Ubuntu up and running the following aplications/configuration/fixes:

ATOM - a great GUI text editor
Automatic login
barrier - great KVM program, free version of synergy, lets you copy and paste between machines
chrome remote desktop - my RDP of choice because the phone app is great for remote admin
fix for checkdisk operation on boot with some M.2 drives
change from wayland to x11 for chrome remote desktop - part of a preventative measure against a login loop error
chrome browser

None of this work is my own, but it took far too many hours of troubleshooting/google-fu to figure out how to get this to run so I figure I would save someone else the headache.

These just happen to be the programs I enjoy using and I know they are not best practice/most secure, I'm sure someone smarter than myself could automate this via chef/ansible or something.

The expected result for my setup was as follows:

 - windows machine = MAIN
 - Ubuntu machine = UBUNTU
 - monitor 1 = MON1
 - monitor 2 = MON2
```sh
  +--------+                                        +--------+
  |        |                                        |        |
  |        |                                        |        |
  |        |   +---------------+  +---------------+ |        |
  | UBUNTU |   |               |  |               | | MAIN   |
  |        |   |  MON2         |  |   MON1        | |        |
  |        |   |               |  |               | |        |
+-+        +---+               +--+               +-+        +----+
| +--------+   +---------------+  +---------------+ +----------   |
|                                                                 |
|    desk                                                         |
+-----------------------------------------------------------------+
| | |                                                         | | |
| | |                                                         | | |
| | |                                                         | | |
| | |                                                         | | |
| | |                                                         | | |
+-+ |                                                         +-+ |
  +-+                                                           +-+
```

 - MAIN on MON1
 - UBUNTU on MON2
 - share single mouse/keyboard on MAIN with UBUNTU via barrier, mouse can go from MON1 to MON2 for local control
 - control UBUNTU on same session shown on MON2 remotely via chrome remote desktop
 - UBUNTU logs in after reboot/power restore starting both barrier and chrome remote desktop


# Step 1
--------------------------------------
#### INSTALL ATOM - [optional - replace commands in this guide containing "atom --no-sandbox" with you alternative]

Do not use the snap from the software center as it will not allow you to edit files with sudo. This will let you use "sudo atom --no-sandbox [/dir/to/file/somefile.extention]" to edit locked files. Less secure, but less of a headache IMO.

Follow the official guide located at:
```sh
https://flight-manual.atom.io/getting-started/sections/installing-atom/#platform-linux
```

# Step 2
---------------------------------------
#### CONFIGURE AUTO-LOGIN - [important - set user to autologin after install not during to avaid an login loop error]

change the autologin conf file
```sh
sudo atom --nosandbox /etc/gdm3/custom.conf
```
change the line
```sh
#WaylandEnable=false 
```
to
```sh
WaylandEnable=false
```

and
```sh
AutomaticLoginEnable=True
```
to
```sh
AutomaticLoginEnable=true
``` 
for some reason it does not like "T" in True instead use "t" true, the capital T is set if you selected this option when you installed Ubuntu

# Step 3a
--------------------------------------
#### BARRIER CLIENT - [optional - used as a KVM for sharing a single mouse & keyboard so you can display your machine on a second monitor]

download the snap from ubuntu software center [see BARRIER ALT below for terminal install steps for automation]

populate the servier IP and make a config and save it (might be able to edit the barrier.desktop file to autopopulate this by changing:
```sh
Exec=barrier > Exec=barrier --no-restart --name [name of computer] --enable-crypto 10.10.10.10 #set this to your barrier server IP
```
 
create a Startup application
```sh
  name:barrier
  command: barrier 
```
check that the .desktop file is in the ~.config folder 

barrier.desktop
```sh
[Desktop Entry]
Type=Application
Exec=barrier
Hidden=true
NoDisplay=false
X-GNOME-Autostart-enabled=true
Name[en_US]=barrier
Name=barrier
Comment[en_US]=start barrier on login
Comment=start barrier on login
```
add barrier to the favorites close it then reopen from favorites bar. Client should be checked, server ip grey with the ip populated, auto-config checked.

# Step 3b
---------------------------------------
#### BARRIER CLIENT ALT

- install barrier from terminal (for automation in future)

Open up a terminal window

Issue the command 
```sh
sudo snap install barrier
```

# Fix 1
---------------------------------------
#### CHECKDISK ON BOOT ISSUE (only do if needed)

edit /etc/default/grub
```sh
sudo atom --no-sandbox /etc/default/grub
```

add parameter fsck.mode=skip before quiet splash then save
```sh
GRUB_CMDLINE_LINUX_DEFAULT="fsck.mode=skip quiet splash" 
```

update grub
```sh
sudo update-grub
```

reboot

# Step 4
---------------------------------------
#### DISABLE WAYLAND

edit the gnome config
```sh
sudo atom --no-sandbox /etc/gdm3/custom.conf
```

uncomment line
```sh
#WaylandEnable=false
```
to look like
```sh
WaylandEnable=false
```
restart gnome
```sh
sudo systemctl restart gdm3
```

# Step 5
---------------------------------------
#### REMOVE WAYLAND

remove wayland from Ubuntu
```sh
sudo apt-get purge --auto-remove gnome-session-wayland
```

reboot

# Step 6
---------------------------------------
#### INSTALL CHROME

download the .deb file (or follow googles instructions)
```sh
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
```

install the file
```sh
sudo apt install ./google-chrome-stable_current_amd64.deb
```

start chrome, log in to your google account

# Step 7
---------------------------------------
#### CHROME REMOTE DESKTOP

install the chrome remote desktop extension for chrome from the google store in chrome

download the .deb
```sh
wget https://dl.google.com/linux/direct/chrome-remote-desktop_current_amd64.deb
```

install the .deb
```sh
apt install /tmp/chrome-remote-desktop_current_amd64.deb
```

make the config directory
```sh
mkdir ~/.config/chrome-remote-desktop
```

stop and start the service
```sh
systemctl stop chrome-remote-desktop
systemctl start chrome-remote-desktop
```

go to the chrome remote desktop site in chrome and the machine should be configurable now

configure computer name and pin

follow the monkey guide below to share same session instead of starting another # (do before reboot to avoid boot loop)

# Step 8
---------------------------------------
#### MONKEY GUIDE FOR SAME SESSION CHROME REMOTE DESKTOP

stop Chrome Remote Desktop (It is OK if it says that the daemon was not currently running)
```sh
/opt/google/chrome-remote-desktop/chrome-remote-desktop --stop
```

backup the original configuration
```sh
sudo cp /opt/google/chrome-remote-desktop/chrome-remote-desktop /opt/google/chrome-remote-desktop/chrome-remote-desktop.orig
```

edit the config file in atom
```sh
sudo atom --no-sandbox /opt/google/chrome-remote-desktop/chrome-remote-desktop
```

find DEFAULT_SIZES and amend to the remote desktop resolution. example
```sh
DEFAULT_SIZES = "1920x1080"
```

find the display number in terminal
```sh
echo $DISPLAY
```

set the X display number to the current display number (that you found in the previous command, mine was :0)
```sh
FIRST_X_DISPLAY_NUMBER = 0
```

comment out sections that look for additional displays
```sh
# while os.path.exists(X_LOCK_FILE_TEMPLATE % display):
# display += 1
```

reuse the existing X session instead of launching a new one. Alter launch_session() by commenting out launch_x_server() and launch_x_session() and instead setting the display environment variable, so that the function definition ultimately looks like the following:
```sh
def launch_session(self, x_args):
self._init_child_env()
self._setup_pulseaudio()
self._setup_gnubby()
#self._launch_x_server(x_args)
#self._launch_x_session()
display = self.get_unused_display_number()
self.child_env["DISPLAY"] = ":%d" % display
```

save and exit the editor

start Chrome Remote Desktop:
```sh
/opt/google/chrome-remote-desktop/chrome-remote-desktop --start
```

on seperate computer login to the remote desktop, if you have the machine hooked up to a monitor, you should be seeing that the remote session is controlling what is on the screen of the monitor

# Step 9
---------------------------------------
#### PREVENT CHROME REMOTE DESKTOP LOGIN LOOP

edit config to add an exit before initialization to prevent chrome remote desktop from starting before login
```sh
sudo atom --no-sandbox /etc/init.d/chrome-remote-desktop
```

add an exit after the beginning of the file, should look like
```sh
#!/bin/bash
exit 0
[ rest of file]
```

save and exit atom

add chrome remote desktop to startup applications (this starts it AFTER the autologin)
```sh
name: chrome remote desktop
command:/opt/google/chrome-remote-desktop/chrome-remote-desktop --start
description: starts CRD after login
```

save and test with a reboot

--------
## FINISHED !
