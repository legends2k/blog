+++
title = "Arch Linux Configuration"
description = "Desktop environment, dual GPU and stuff"
date = "2018-05-27T17:00:46-07:00"
tags = ["tech", "tools", "linux"]
draft = true
+++

# Network

Once inside the newly installed Arch, I noticed that there's no network.  To enable redo _§2 Wireless Network_ above.  I did it only to realize that for _netctl_ to hook to a WPA-secured network, `wpa_supplicant` is needed but absent in the installed system.  Since there's no network, I'd to reboot into the installation USB, setup network and install it from there to the new OS by chrooting

```
root@archiso / # mount /dev/lvmg1/root /mnt
root@archiso / # mount /dev/nvme0n1p5 /mnt/boot
root@archiso / # mount /dev/lvmg1/home /mnt/home
root@archiso / # mount /dev/sda5 /mnt/var
root@archiso / # swapon /dev/sda6
root@archiso / # arch-chroot /mnt
[root@archiso /]# pacman -S wpa_supplicant
[root@archiso /]# exit
root@archiso / # reboot
```

To permanently enable a network on boot, enable the service

```
netctl enable infoprobe
```

If on every boot you get this with a long wait

> [***      ] A start job is running for dhcpcd on wlp3s0 (14 s / 1min 30s). 

As [discussed in the forums](https://bbs.archlinux.org/viewtopic.php?id=213363) set `/etc/systemd/system/systemd-user-sessions.service`

```
[Unit]
Description=Permit User Sessions
Documentation=man:systemd-user-sessions.service(8)
After=remote-fs.target nss-user-lookup.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/lib/systemd/systemd-user-sessions start
ExecStop=/usr/lib/systemd/systemd-user-sessions stop
```

# Enlisting Windows

Make sure you've the `os-prober` package installed.  Mount the EFI partition and re-run GRUB config maker

```
mount /dev/nvme0n1p1 /boot/efi
grub-mkconfig -o /boot/grub/grub.cfg
```

It should list Windows as one of the options now.

# Desktop Environment

To have Xfce up and running, we need a display system (_Xorg_; Xfce can’t run atop _Wayland_ yet) and a display manager (_LXDM_)

```
pacman -Syy
pacman -S --needed xorg xorg-server xfce4 xfce4-goodies lxdm xf86-input-synaptics
```

Once done, enable (for future boots) and start the display manager

```
systemctl enable lxdm.service
systemctl start lxdm.service
```

For the first run, make sure to set the _Session_ and _Locale_; not doing so led to a login screen loop.

# Display

I got infinite waits every time I shutdown due to Nouveau drivers for the Nvidia; for this reason I disabled `lxdm.service` and operated from the terminal.  First do

```
lspci -k | grep -A 2 -E "(VGA|3D)"
```

to be sure of the graphic devices you have.

## Intel

To get Intel graphics working

```
pacman -S --needed xf86-video-intel mesa mesa-demos
```

With Skylake and newer processors (Kabylake, …) we can enable `i915` module for [early KMS start](https://wiki.archlinux.org/index.php/Kernel_mode_setting#Early_KMS_start) in `/etc/mkinitcpio.conf` and run `mkinitcpio -p linux`

```
# MODULES
# …
MODULES=(i915)
```

Also enable GuC, HuC and FBC by setting `/etc/modprobe.d/i915.conf`

```
options i915 enable_guc=-1 enable_fbc=1
```

Screen tearing when scrolling large walls of text in Firefox is a common occurrence.  To fix this set `/etc/X11/xorg.conf.d/20-intel.conf`

```
Section "Device"
    Identifier "Intel Graphics"
    Driver "intel"
    Option "TearFree" "true"
EndSection
```

The Intel device needs only these but still I was unable to `startx` or `systemctl start lxdm.service`; even if it shows up, it usually hung during power down sequence.  All because the Nvidia device was preferred over Intel's and its drivers are broken.

## Nvidia

Switching between Nouveau and proprietary drivers (both `nvidia` and `nvidia-lts`) I tried mucking around with `/etc/X11/xorg.conf`, created and tweaked `/etc/X11/xorg.conf.d/20-nvidia.conf`, ran `nvidia-xconfig` and `nvidia-settings`, , but until I got [Bumblebee](https://bumblebee-project.org/) nothing worked for me.

Remove all nouveau-related and install Nvidia supplied packages.  Install Bumblebee and friends:

```
pacman -Rs nouveau xf86-video-nouveau libvdpau
pacman -S nvidia nvidia-utils
pacman -S bumblebee primus bbswitch
```

In order to use Bumblebee, add user to the `bumblebee` group

```
gpasswd -a root bumblebee
```

Start service to see if things work as expected and enable it for persistence

```
systemctl start bumblebeed.service
systemctl enable bumblebeed.service
```

Once all the setting up is done, do

```
glxinfo | grep "OpenGL Renderer"
glxgears -info
optirun glxgears -info
```

Running _glxgears_ with and without `optirun` should show the right GPU selected for running the demo.

# Draft

* Pulseaudio
* Bluebooth - bluez, blueman
    - Auto Power ON: [disable in Power Manager plug-in](https://www.linux.com/forums/networking/solved-bluez-543-have-bluetooth-disabled-boot)
        - TODO: update this in [Arch Wiki](https://wiki.archlinux.org/index.php/Bluetooth#Auto_power-on_after_boot)
    - Obex push folder: default says `Root` which is `~/.cache/obex`, set it to something meaningful in _Local services_ -> _Transfer_
    - [Avoid root password prompt](https://wiki.archlinux.org/index.php/Blueman#Permissions) at every login with a polkit rules file
    - Add yourself to `lp` group
* Font config
    - Adapta theme package suggests a couple of fonts which seems interesting
* Archive manager
    - Install packages: `p7zip`, `unzip`, `unrar`
    - GUI `xarchiver` which should integrate with Thunar; [issue](https://github.com/ib/xarchiver/issues/62) of preferences not persistant, fixed by a workaround
* Lock screen
    - `xlockmore` works fine, just be aware that switching to TTY (with Ctrl + Alt + 2, ...) is still possible with it
    - Integrates seamlessly with `xflock4` a script (`/usr/bin/xflock4`) which tries different lockers
    - Other lockers have issues
        - Don’t work (e.g. _gnome-screensaver_)
        - Is not in the official repos (e.g. _sflock_ and also didn’t work)
        - Only `root` can unlock (e.g. _physlock_)
        - Need a DM (e.g. _light-lock_)
* Setting system-wide monospace font through _Appearance_ setting dialog neither fixes Terminal or Mousepad’s fonts; overriding each manually seems silly
    - `gsettings set org.gnome.desktop.interface monospace-font-name 'Ubuntu Mono 16'` does the trick for Terminal
    - Customizing font config per-user: `~/.config/fontconfig/fonts.conf`
    - See [here](https://unix.stackexchange.com/questions/106070/changing-monospace-fonts-system-wide) for both options
* [Disable desktop zoom](https://forum.xfce.org/viewtopic.php?pid=41556#p41556) in Xfce4 with _Settings Editor_ → _xfwm4_, uncheck `zoom_desktop`
* Traditionally RTC (h/w clock that doesn’t understand time standards) is set in local time; Windows reads it as local by default.  Linux doesn’t, as it recommends setting it in GMT and let the OS services deal with time zone and DST variations.  Forcing Linux with `timedatectl set-local-rtc 1` is possible.  However, `man timedatectl` warns that it will create problems when changing time zones and DST changes; one has to rely on booting into Windows at least twice annually (in Spring and Fall -- [see here](https://unix.stackexchange.com/q/234689/30580)) for DST adjustments.  Setting RTC in GMT (like macOS) is appropriate; a registry change + restart on Windows will make it read RTC as GMT too.  This is the recommended way of setting time in dual boot machines.  For time sync with NTP, do `timedatectl set-ntp true`.  Do `timedatectl status` to check if everything is OK:

```
               Local time: Thu 2018-10-18 16:04:49 IST
           Universal time: Thu 2018-10-18 10:34:49 UTC
                 RTC time: Thu 2018-10-18 10:34:49
                Time zone: Asia/Kolkata (IST, +0530)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

* To make packages from AUR install `base-devel` package, make `sudo` group, add user to it
```
su
visudo                      # uncomment sudo group
groupadd sudo               # if not already existing
gpasswd -a sundaram sudo
```

`makepkg -si` needs `sudo` to install the built package.

1. To enable Tamil in browser (and other interfaces), I’d to do

```
pacman -S firefox-i18n-ta 
git clone https://aur.archlinux.org/ttf-tamil.git
pacman -S base-devel
makepkg -si
```

2. Install `udisks2` for auto-mounting NTFS partitions and removable (USB) disks.  To allow users of group `sudo` to mount NTFS drives (with write permissions) without asking for `root` password, [enable the group in _polkit_](https://unix.stackexchange.com/a/207667).  I like using _udisks2_ over `/etc/fstab`; the former is intelligent, has permissions and mount points that are user-based; the latter is plain hard-coding.  A recommended GUI, if needed, is [_udiskie_](https://github.com/coldfix/udiskie).
3. For screenshots, Xfce has `xfce4-screenshooter` which has to be hooked to <kbd>PrintScr</kbd> through `xfce4-keyboard-settings` under the _Application Shortcuts_ tab.
4. `laptop-mode-tools` seems to be an important package to be installed for power saving on battery in a laptop.  Also install `hdparm` and `cpupower`.
    + Start with `sudo systemctl enable laptop-mode.service`
    + In `/etc/laptop-mode/conf.d/intel-sata-powermgmt.conf` set `CONTROL_INTEL_SATA_POWER="1"`
    + In `/etc/laptop-mode/conf.d/lcd-brightness.conf`, set `BRIGHTNESS_OUTPUT="/sys/class/backlight/intel_backlight/brightness"` and `CONTROL_BRIGHTNESS=1`
    + Set `BATT_HD_POWERMGMT=200` and `LM_AC_HD_POWERMGMT=240` in `/etc/laptop-mode/laptop-mode.conf`.  Verify if this is in action with `hdparm -B /dev/sda`
5. For natural scrolling with laptop’s touchpad, similar to iPad, enabling _Reverse scroll direction_ under _Mouse and Touchpad_ settings fixes in most places except in _Terminal_.
6. _Edit with Emacs_ to use `emacsclient` see https://emacs.stackexchange.com/q/14055/4106

# Issues

1. Log out and in mouse cursor frozen
2. Screen locker with tty security but also works with Xfce4
3. Scroll neutralizing at all places


# References

1. Arch Linux Installation Guide
2. [Installing Arch Linux with LVM](https://gilyes.com/Installing-Arch-Linux-with-LVM/) by George Ilyes
3. [Multi HDD/SSD Partitioning Scheme](https://wiki.debian.org/Multi HDD/SSD Partition Scheme)
4. [GRUB](https://wiki.archlinux.org/index.php/GRUB#Check_for_an_EFI_System_Partition) on Arch Linux Wiki
5. [Dual-boot with Windows](https://wiki.archlinux.org/index.php/Dual_boot_with_Windows#UEFI_systems)
6. [Pick A Suitable Desktop Environment For Arch Linux](https://www.2daygeek.com/install-xfce-mate-kde-gnome-cinnamon-lxqt-lxde-budgie-deepin-enlightenment-desktop-environment-on-arch-linux/)

