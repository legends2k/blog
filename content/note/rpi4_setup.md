+++
title = "Raspberry Pi 4 Setup"
description = "Access via SSH (Headless)"
date = 2021-02-09T11:27:59+05:30
tags = ["tech", "hardware"]
+++

I neither have a micro HDMI adapter nor a dedicated display for the Pi.  I just want to use it as a server.  Thanks to Raspbian for providing a simple way to auto-enable SSH on boot!

I prefer the leaner, manual approach over [NOOBS][], the more popular option.

1. [`dd` Raspbian’s image][dd-rpi4-iso] on to a new SD card
2. **To auto-start SSH service on boot `touch ssh` in `/boot` partition**
3. Insert SD card, connect ethernet cable (RJ-45) and boot
    - Check IP address assigned to RPi from your router’s portal
4. `ssh -l pi IP`
    - Change default password (`raspberry`) with `passwd`
    - `sudo apt update`
    - `sudo apt upgrade`
5. Use `raspi-config` and setup
    - Locale
    - Wi-Fi country code
      + Needed to abide by your country’s wireless laws
      + Radio device may not work properly if not done
    - Wi-Fi (SSID and password)
    - Enable SSH (for good measure)
    - Check `systemctl is-enabled sshd`
    - Make sure wireless isn’t soft-blocked: `rfkill list` and `rfkill unblock wifi`
6. Unplug ethernet and wireless should work now
    - Use `ifconfig` to know your Mac address
    - `eth0`/`wlan0` interfaces that’re up have _inet_ (IP address) populated
7. Use `raspi-config` and enable VNC
    - Setup VNC password: `sudo vncpasswd -service`
    - Boot into _Desktop_ instead of _Console_ in _System Options -> Boot / Auto Login_
      + See this if you want to [boot to CLI and `startx` later \[3\]](#references)
    - Make sure to select a resolution under _Display Options -> Resolution_
    - For macOS _Screen Sharing_ to work, make `/etc/vnc/config.d/common.custom` with `Authentication=VncAuth`
    - `sudo reboot -n`
    - Check VNC server status: `systemctl status vncserver-x11-serviced`
    - You should be able to VNC to your Pi by first giving the VNC password and then `pi`’s password
8. Install Firefox: `sudo apt install firefox-esr`
    - `sudo update-alternatives --config x-www-browser`

Another interesting alternative to Raspbian is [DietPi][] (also Debian based), comes with a bunch of software optimized for the Pi.

[dd-rpi4-iso]: https://www.raspberrypi.org/documentation/installation/installing-images/linux.md
[NOOBS]: https://github.com/raspberrypi/noobs
[DietPi]: https://dietpi.com/


# References

1. [SSH (Secure Shell)](https://www.raspberrypi.org/documentation/remote-access/ssh/README.md)
2. [The Ultimate Guide to the Raspi-Config Tool](https://pimylifeup.com/raspi-config-tool/)
3. [Headless VncServer Configuration](https://raspberrypi.stackexchange.com/a/79626)
4. [How to start GUI with startx command](https://raspberrypi.stackexchange.com/q/84804)

