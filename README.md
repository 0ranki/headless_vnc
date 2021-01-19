### How to set up a Linux box with a persistent session and VNC access
This is how I managed to set up a headless Linux server with VNC access. Instructions are adapted from [these instructions on Steam](https://steamcommunity.com/sharedfiles/filedetails/?id=680514371), [x11vnc man page](https://linux.die.net/man/1/x11vnc) and various tutorials/blog posts online

You will get a single always-on local desktop session to which you can login via VNC. No multiple sessions or users!

### Prerequisites:
- A linux system with a working graphical environment on <b>X11</b>
  - install the system with a monitor, keyboard and mouse plugged in. You'll need a way to generate the edid.txt file and `/etc/X11/xorg.conf`. I had an NVIDIA card so I used `nvidia-settings` to do that. On AMD/Intel graphics you should be able to use `sudo Xorg -configure` (https://www.cyberciti.biz/tips/create-a-xorgconf-file.html), but I haven't tried this. You can also use the edid.txt provided in the Steam instructions link above, there are options for 1920x1080 and a 1440x900 monitors. Copy and paste the EDID informations to e.g. `/etc/X11/edid.txt`.
- `x11vnc` installed
- You can unplug the monitor and peripherals after you have generated the `xorg.conf` and `edid.txt`
- an SSH server running is a good idea so you have some other way of accessing the headless box
- I've tried this with Xfce, MATE, LXQt and GNOME, so I assume most DEs will work, but the experience is better on a lighter DE with few or no animations
- I've tested on Fedora, CentOS, Ubuntu and Ubuntu MATE, and the above instructions (Steam) are for Arch Linux, so most distros will probably work. There might be slight differences you'll have to figure out yourself

### Acquiring `xorg.conf` and `edid.txt` on NVIDIA
- Install the NVIDIA proprietary drivers, log in to an Xorg session of your chosen desktop environment. Open a terminal and run `sudo nvidia-settings` (sudo is required so you can generate `/etc/X11/xorg.conf`
- Generate `/etc/X11/xorg.conf`:
  - from "X Server Display Configuration, click "Save to X configuration file" and set the path `/etc/X11/xorg.conf`
- Generate `/etc/X11/edid.txt`: (or use the provided one, you can also save this anywhere you like)
  - From the list of ports, choose the one your monitor is connected to (for me it was DVI-I-1) and click "Acquire EDID", then select EDID File Format as ASCII. Save the file somewhere, I used `/etc/X11/edid.txt`
- Edit `/etc/X11/xorg.conf` and in the `<Device>` section add the lines:
  ```
  Option "ConnectedMonitor" "<display>"
  Option "CustomEDID" "<display>:<path>/edid.txt"
  ```
  Replace `<display>` and `<path>` with yours, for me it was `Option "ConnectedMonitor" "DVI-I-1"` and `Option "CustomEDID" "DVI-I-1:/etc/X11/edid.txt"`
- Save the file
- Edit (create) `/etc/X11/Xwrapper.config`:
  - add the line `allowed_users=anybody`
  - this will allow the X session to be started anywhere, It seems this is necessary to start X without an attached display. You can try without it too.

### Make GDM use Xorg instead of Wayland:
- edit `/etc/gdm/custom.conf` and uncomment the line `WaylandEnable = false`

### Set up automatic login: [Arch wiki](https://wiki.archlinux.org/index.php/GDM#Automatic_login)
- For GDM: (Ubuntu)
  - Under the `[Daemon]` section, add
  ```
  AutomaticLogin=username
  AutomaticLoginEnable=True
  ```
  - replace `username` with your username
  - edit `/var/lib/AccountsService/users/<username>` and make sure the `XSession=<your preferred DE>` is correct. The possible sessions are in `/usr/share/xsessions`, use one of those without the `.desktop` extension. This should be correct if you have only one DE installed.

- For SDDM: [Arch wiki](https://wiki.archlinux.org/index.php/SDDM#Autologin)
  - create the file `/etc/sddm/sddm.conf.d/autologin.conf` with the following content:
  ```
  [Autologin]
  User=<username>
  Session=<session>.desktop
  ```
  - replace `<username>` and `<session>` with your values

- For other DMs you can find instructions online

### Test that everything is working so far
- Reboot and see that you get logged in correctly to an Xorg session with the display still connected

### Set up `x11vnc`
- install `x11vnc` if you haven't already
- Generate a password for the VNC session:
  - run `x11vnc -storepasswd` as your regular user
  - <b>Please note that VNC is by default unencrypted! If you use this over the internet, I suggest using a VPN connection or tunneling over SSH</b>
  - You can also set up the VNC to be encrypted, but I used a VPN (Wireguard)
- Test:
  - run `sudo x11vnc -auth passwd
