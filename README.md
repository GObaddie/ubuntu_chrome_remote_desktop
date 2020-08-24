# ubuntu_chrome_remote_desktop
## How to get Barrier and Chrome Remote Desktop working in Ubuntu 20.04 (Focal Fossa)

This is a step-by-step guide to get Ubuntu up and running with a few helpful aplications/configuration/fixes:

Automatic login
Atom - a great GUI text editor
Barrier - a great KVM program (like Synergy, but free!). Lets you copy and paste between machines.
Chrome Remote Desktop - my RDP of choice because the phone app is great for remote admin.
Fix for checkdisk operation on boot with some M.2 drives.
Change from wayland to x11 for chrome remote desktop - part of a preventative measure against a login loop error
Chrome browser

This guide is structured in such a way that you can choose some or all of these changes at your discretion. These just happen to be the programs I enjoy using and I know they are not best practice/most secure, I'm sure someone smarter than myself could automate this via chef/ansible or something.

None of this work is my own, but it took far too many hours of troubleshooting/google-fu to figure out how to get this to run so I figure I would save someone else the headache.

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

As an alternative, you can use pico (a command-line editor which Ubuntu ships with) by running 
```sh
sudo pico file-name-here
```

# Step 2
---------------------------------------
#### CONFIGURE AUTO-LOGIN

**During the initial Ubuntu installation, DO NOT choose the auto login option.** You'll have automatic login working at the end of this guide, but it's important you start with manual login to avoid a login loop error.

In the autologin conf file
```sh
sudo atom --no-sandbox /etc/gdm3/custom.conf
```
uncomment the line
```sh
#WaylandEnable=false 
```
to
```sh
WaylandEnable=false
```
and
```sh
#AutomaticLoginEnable=true
```
to
```sh
AutomaticLoginEnable=true
```
You'll also need to specify the user by uncommenting the next line and adding your username;
```sh
AutomaticLogin=usernamehere
```

If you enabled automatic login during install, you might be able to recover things at this point. For some reason, the installer will set AutomaticLoginEnabled to True instead of true (note capital T). It really shouldn't matter, but correcting the case (to lower case) might save you a re-install down the line.

After you've installed Chrome, you might get a keyring popup every time you reboot. If this happens, launch Passwords and Keys from the app menu; you might need to hit the "back" arrow in the upper left, but you're looking for a screen that reads "Passwords and Keys" with a "Login" folder. Right click on that folder, and choose Change Password; enter your old password, and make your new password blank. This has some security implications so read the warning that pops up carefully. The password manager might crash on your next reboot, but this will be a one off - on subsequent reboots you should be into Ubuntu smoothly with no extra passwords / clicking around required.

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
#### CHECKDISK ON BOOT ISSUE (Might not be an issue - skip this if you're not seeing errors)

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

You should have done this earlier during auto login setup, but double check Wayland is disabled:

```sh
sudo atom --no-sandbox /etc/gdm3/custom.conf
```

This line should be uncommented:
```sh
WaylandEnable=false
```
Restart gnome so it takes effect.
```sh
sudo systemctl restart gdm3
```


# Step 5
---------------------------------------
#### INSTALL CHROME

Download the .deb file (or follow Google's instructions)
```sh
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
```

install the file
```sh
sudo apt install ./google-chrome-stable_current_amd64.deb
```

Start Chrome, log in to your Google account.

# Step 6
---------------------------------------
#### CHROME REMOTE DESKTOP

Install the [Chrome Remote Desktop extension](https://chrome.google.com/webstore/detail/chrome-remote-desktop/inomeogfingihgjfjlpeplalcfajhgai) in Chrome.


Download the .deb for the client:

```sh
wget https://dl.google.com/linux/direct/chrome-remote-desktop_current_amd64.deb
```

Install it:

```sh
apt install ./chrome-remote-desktop_current_amd64.deb
```

Make the config directory:
```sh
mkdir ~/.config/chrome-remote-desktop
```

Stop and start the service:
```sh
systemctl stop chrome-remote-desktop
systemctl start chrome-remote-desktop
```

Go to the [Chrome Remote Desktop site](https://remotedesktop.google.com/access/) in Chrome and the machine should be configurable now (press "Turn On" and follow the prompts).

Configure computer name and PIN. If it stays on the final setting up stage for more than thirty seconds, try reloading the page - it might have taken effect.

**Important**: follow the monkey guide below to share the same session instead of starting another #. Do this before reboot to avoid a boot loop.

# Step 7
---------------------------------------
#### MONKEY GUIDE FOR SHARED SESSION IN CHROME REMOTE DESKTOP

Ubuntu supports multiple display sessions, and Chrome Remote Desktop will (by default) leverage this feature. That means you can be connected on the machine itself, and have several applications open; when you connect over remote desktop, it will start a new session (without your existing state). Conversely, if you start doing something remotely, then try to finish it up on the machine locally, all the apps you had open won't appear on the local display. As well as being a bit annoying, this can cause all sorts of nasty bugs (e.g the most recent state in one session clobbering the other during shutdown; launching applications in one session and they actually appear in the other... it's a real mess). Follow these steps to override the "smart" functionality, and just have a single session that's shared between local and remote access.

*There are probably some very clever reasons to run it the default way, and changing it like this is less secure - for example, if you unlock the machine remotely over RDP, the machine unlocks on the local session too - someone with physical access could see your mouse moving around, watch what you were typing or even take over with a keyboard / mouse. That said, I just can't get it to work at all in the default setup, so "less secure and working" is good enough for me.*.

Stop Chrome Remote Desktop (It is OK if it says that the daemon was not currently running)
```sh
/opt/google/chrome-remote-desktop/chrome-remote-desktop --stop
```

Backup the original configuration
```sh
sudo cp /opt/google/chrome-remote-desktop/chrome-remote-desktop /opt/google/chrome-remote-desktop/chrome-remote-desktop.orig
```

Edit the config file
```sh
sudo atom --no-sandbox /opt/google/chrome-remote-desktop/chrome-remote-desktop
```

Find DEFAULT_SIZES and amend to the remote computer's resolution. Example:
```sh
DEFAULT_SIZES = "1920x1080"
```

Find the local display number in Terminal:
```sh
echo $DISPLAY
```

Set the X display number to the current display number (that you found in the previous command, mine was :0)
```sh
FIRST_X_DISPLAY_NUMBER = 0
```

Comment out sections that look for additional displays:
```sh
# while os.path.exists(X_LOCK_FILE_TEMPLATE % display):
# display += 1
```

Reuse the existing X session instead of launching a new one. Alter launch_session() by commenting out launch_x_server() and launch_x_session() and instead setting the display environment variable, so that the function definition ultimately looks like the following:
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

Save and exit the editor.

Start Chrome Remote Desktop:
```sh
/opt/google/chrome-remote-desktop/chrome-remote-desktop --start
```

On a seperate computer, login to the remote desktop. If you have the host machine hooked up to a monitor, you should be seeing that the remote session is controlling what ever's on the screen of the local monitor.

##### A note on headless setups

Once you've got your computer reliably, remotely accessible, you might be tempted to remove the monitor (especially if it's a media server or similar that you very rarely use locally). This can cause problems - Linux is smart enough to check if a monitor is connected as it starts up. If there's nothing there, it won't output any video or start a display session - which is what you need to connect remotely. The good news is this is a problem for crypto farmers too, so there are lots of little gadgets you can make or buy that plug in to the video port and trick the computer (at a hardware level) into believing there's a screen attached. This means a display session will spin up at boot time, which you can in turn connect to.

You should do your own research, but my attempts to DIY by shorting pins on a VGA adapter with resistors didn't trick my motherboard. I ended up buying a headless HDMI Dummy Plug - lots of options on eBay, mine cost about 5 AUD.


# Step 8
---------------------------------------
#### PREVENT CHROME REMOTE DESKTOP LOGIN LOOP

Edit the config to add an early exit before initialization; this prevents Chrome Remote Desktop from starting before login:
```sh
sudo atom --no-sandbox /etc/init.d/chrome-remote-desktop
```

All you need to do is add an exit after the beginning of the file, should look like this:
```sh
#!/bin/bash
exit 0
[ rest of file]
```

Save and exit your editor.

Add Chrome Remote Desktop to startup applications (just search Startup Applications from the app menu and add a new entry). This starts it AFTER the autologin.
```sh
name: chrome remote desktop
command:/opt/google/chrome-remote-desktop/chrome-remote-desktop --start
description: starts CRD after login
```

Save and test with a reboot.

--------
## FINISHED !
