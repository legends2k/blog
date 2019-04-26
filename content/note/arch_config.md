+++
title = "Arch Linux Configuration"
description = "Desktop environment, dual GPU and stuff"
date = "2018-05-27T17:00:46-07:00"
publishdate = "2019-04-26T11:31:00+05:30"
tags = ["tech", "tools", "linux"]
+++

# Network

In the newly installed Arch you might notice that there's no network connectivity.  To enable do what you did during installation (see _Arch Linux Installation_, [§2 Wireless Network][]).  I did it only to realize, for `netctl` to hook to a WPA-secured network, the `wpa_supplicant` package is needed but was absent on the installed system.  To connect to the internet, you need a package from the internet!?  To get out of this Catch 22 situation, I'd to reboot from the installation USB, setup network and install `wpa_supplicant` to the new OS by chrooting

{{< highlight basic >}}
root@archiso / # mount /dev/lvmg1/root /mnt
root@archiso / # mount /dev/nvme0n1p5 /mnt/boot
root@archiso / # mount /dev/lvmg1/home /mnt/home
root@archiso / # mount /dev/sda5 /mnt/var
root@archiso / # swapon /dev/sda6
root@archiso / # arch-chroot /mnt
[root@archiso /]# pacman -S wpa_supplicant
[root@archiso /]# exit
root@archiso / # reboot
{{< /highlight >}}

To permanently enable a network on boot, enable the service

{{< highlight basic >}}
netctl enable infoprobe
{{< /highlight >}}

If on every boot you get this with a long wait

{{< highlight basic >}}
[***      ] A start job is running for dhcpcd on wlp3s0 (14 s / 1min 30s).
{{< /highlight >}}

As [discussed in the forums][dhcp_job] set `/etc/systemd/system/systemd-user-sessions.service`

{{< highlight cfg >}}
[Unit]
Description=Permit User Sessions
Documentation=man:systemd-user-sessions.service(8)
After=remote-fs.target nss-user-lookup.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/lib/systemd/systemd-user-sessions start
ExecStop=/usr/lib/systemd/systemd-user-sessions stop
{{< /highlight >}}

According to Arch Wiki’s [domain name resolution][]

> The `Glibc` resolver provides only the most basic necessities, it does not cache queries nor provides any security features. If you require more functionality, use another resolver.

However, another statement admonitions that a router usually does this caching at the network-level, so you can skip setting up a more robust resolver.

[§2 Wireless Network]: {{< relref "arch_install#wireless-network" >}}
[dhcp_job]: https://bbs.archlinux.org/viewtopic.php?id=213363
[domain name resolution]: https://wiki.archlinux.org/index.php/Domain_name_resolution#Lookup_utilities

# Resurrecting Windows

On a dual boot setup, if you notice that _Windows_ is’t an option in the boot menu, install the `os-prober` package.  Mount the EFI partition and re-run GRUB config maker:

{{< highlight basic >}}
mount /dev/nvme0n1p1 /boot/efi
grub-mkconfig -o /boot/grub/grub.cfg
{{< /highlight >}}

_Windows_ would come up as an option now, as the prober would be able to _see_ the Windows installation.

# Real Time Clock Setup

Traditionally RTC --- h/w clock that doesn’t understand time standards --- is set in local time; Windows reads it as local by default; Linux doesn’t.  It recommends setting it in GMT and let the OS services deal with time zone and DST variations.  Forcing Linux with `timedatectl set-local-rtc 1` is possible.  However, `man timedatectl` warns that it will create problems when changing time zones and DST changes; one has to rely on booting into Windows [at least twice annually][Linux localtime sync] (in Spring and Fall) for DST adjustments.  Setting RTC in GMT seems appropriate (macOS does this too).  [A registry change][UTC in Windows] + restart will make Windows read RTC as GMT too.  This is the recommended way of setting time in dual boot machines.  For time sync to an [NTP][] server, do `timedatectl set-ntp true`.  Do `timedatectl status` to check if everything is OK:

{{< highlight basic "hl_lines=7" >}}
               Local time: Thu 2018-10-18 16:04:49 IST
           Universal time: Thu 2018-10-18 10:34:49 UTC
                 RTC time: Thu 2018-10-18 10:34:49
                Time zone: Asia/Kolkata (IST, +0530)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
{{< /highlight >}}

[NTP]: https://en.wikipedia.org/wiki/Network_Time_Protocol
[Linux localtime sync]: https://unix.stackexchange.com/q/234689/30580
[UTC in Windows]: https://wiki.archlinux.org/index.php/System_time#UTC_in_Windows

# Desktop Environment

To have _Xfce_ up and running, you need a display system (_Xorg_; Xfce can’t run atop _Wayland_ yet) and a display manager (_LXDM_)

{{< highlight basic >}}
pacman -Syu
pacman -S --needed xorg xorg-server xfce4 xfce4-goodies lxdm xf86-input-synaptics
{{< /highlight >}}

Once done, enable (for future boots) and start the display manager

{{< highlight basic >}}
systemctl enable lxdm.service
systemctl start lxdm.service
{{< /highlight >}}

For the first run, make sure to set the _Session_ and _Locale_; not doing so led to a login screen loop.

For natural scrolling with laptop’s touchpad, similar to iPad, enabling _Reverse scroll direction_ under _Mouse and Touchpad_ settings; this fixes scroll in most places except _Terminal_ 😬.

For sanity, [disable desktop zoom][] "feature" in  _Settings Editor_ → _xfwm4_; uncheck `zoom_desktop`.

[disable desktop zoom]: https://forum.xfce.org/viewtopic.php?pid=41556#p41556

# Display

I got infinite waits every time I tried `poweroff -n` due to Nouveau drivers for Nvidia; for this reason I disabled `lxdm.service` and operated from the terminal.  To list the graphics devices you’ve, do

{{< highlight basic >}}
lspci -k | grep -A 2 -E "(VGA|3D)"
{{< /highlight >}}

## Intel

To get Intel graphics working

{{< highlight basic >}}
pacman -S --needed xf86-video-intel mesa mesa-demos
{{< /highlight >}}

With Skylake and its successors (Kabylake, …) we can enable `i915` module for [early KMS start] in `/etc/mkinitcpio.conf` and run `mkinitcpio -p linux`

{{< highlight cfg >}}
# MODULES
# …
MODULES=(i915)
{{< /highlight >}}

Enable GuC, HuC and FBC in `/etc/modprobe.d/i915.conf`

{{< highlight basic >}}
options i915 enable_guc=-1 enable_fbc=1
{{< /highlight >}}

Screen tearing when scrolling large walls of text in Firefox is a common occurrence.  To fix this set `/etc/X11/xorg.conf.d/20-intel.conf` to

{{< highlight basic >}}
Section "Device"
    Identifier "Intel Graphics"
    Driver "intel"
    Option "TearFree" "true"
EndSection
{{< /highlight >}}

The Intel device needs only these but still I couldn’t `startx` or `systemctl start lxdm.service`; even if stuff shows up, it usually hung during power down sequence.  All because the Nvidia device was preferred over Intel's and its drivers were broken.

[early KMS start]: https://wiki.archlinux.org/index.php/Kernel_mode_setting#Early_KMS_start

## Nvidia

Switching between Nouveau and proprietary drivers (both `nvidia` and `nvidia-lts`), mucking around with `/etc/X11/xorg.conf`, creating and tweaking `/etc/X11/xorg.conf.d/20-nvidia.conf`, running `nvidia-xconfig` and `nvidia-settings` --- none of these worked!  Enter [Bumblebee][] and voilà!

Remove all nouveau-related and install Nvidia-supplied packages.  Install Bumblebee and friends:

{{< highlight basic >}}
pacman -Rs nouveau xf86-video-nouveau libvdpau
pacman -S nvidia nvidia-utils
pacman -S bumblebee primus bbswitch
{{< /highlight >}}

Make sure at least these entries are set right in `/etc/bumblebee/bumblebee.conf`:

{{< highlight cfg >}}
[bumblebeed]
ServerGroup=bumblebee
Driver=nvidia

[driver-nvidia]
KernelDriver=nvidia
{{< /highlight >}}

In order to use Bumblebee, add user to `bumblebee` group

{{< highlight basic >}}
gpasswd -a root bumblebee
{{< /highlight >}}

Start the _Bumblebee_ service to see if things work as expected; enable for service start across reboots.

{{< highlight basic >}}
systemctl start bumblebeed.service
systemctl enable bumblebeed.service
{{< /highlight >}}

Once all the setting up is done, you could choose between the GPUs when running a process.  `optirun` the process to choose Nvidia; the default would be Intel:

{{< highlight basic >}}
glxinfo | grep "OpenGL Renderer"
glxgears -info
optirun glxgears -info
{{< /highlight >}}

Running _glxgears_ with and without `optirun` should show the right GPU selected for the demo running.

Once in a while, you might get this error when `optirun`nig something

{{< highlight basic >}}
[  146.868249] [ERROR]Cannot access secondary GPU - error: [XORG] (EE) NVIDIA(GPU-0): Failed to initialize the NVIDIA GPU at PCI:1:0:0.  Please
[  146.868250] [ERROR]Aborting because fallback start is disabled.
{{< /highlight >}}

The [fix in ArchWiki][failed GPU init] says

{{< highlight basic >}}
su
echo 1 > /sys/bus/pci/devices/0000:01:00.0/remove
echo 1 > /sys/bus/pci/rescan
{{< /highlight >}}

However, since Linux kernel version 5 I didn’t have to do this once!

[Bumblebee]: https://bumblebee-project.org/
[failed GPU init]: https://wiki.archlinux.org/index.php/Bumblebee#Failed_to_initialize_the_NVIDIA_GPU_at_PCI:1:0:0_(Bumblebee_daemon_reported:_error:_[XORG]_(EE)_NVIDIA(GPU-0))

# (Pulse) Audio

Just install `pulseaudio`, `pulseaudio-alsa`, `pavucontrol` and `xfce4-pulseaudio-plugin`; little to no configuring seems to be needed to get audio working.  I set _Built-in Audio_ to _Analog Stereo Output_ since this laptop has stereo out only.

# Bluetooth

Install `bluez`, `bluez-utils` and `blueman` -- for a decent GTK+ bluetooth manager with an applet.  Verify if bluetooth hardware is not hard blocked but is soft blocked.

{{< highlight basic >}}
> rfkill list highlight
1: ideapad_bluetooth: Bluetooth
	Soft blocked: yes
	Hard blocked: no
3: hci0: Bluetooth
	Soft blocked: yes
	Hard blocked: no
{{< /highlight >}}

As mentioned in ArchWiki

> By default the bluetooth daemon will only give out bnep0 devices to users that are a member of the `lp` group.

Create and add yourself to it.

{{< highlight basic >}}
groups $USER            # check if user already in lp
groupadd lp
useradd -G lp $USER
{{< /highlight >}}

Enable and start `bluetooth.service`.

{{< highlight basic >}}
systemctl --now enable bluetooth.service
{{< /highlight >}}

Though this is enough, at every login, there’d be an annoying prompt for the root password, to enable bluebooth.  Add this to `/etc/polkit-1/rules.d/51-blueman.rules` to not prompt for users in the `sudo` group as [detailed in the Wiki][Blueman_permissions]:

{{< highlight cfg >}}
/* Allow users in sudo group to use blueman feature requiring root without authentication */
polkit.addRule(function(action, subject) {
    if ((action.id == "org.blueman.network.setup" ||
         action.id == "org.blueman.dhcp.client" ||
         action.id == "org.blueman.rfkill.setstate" ||
         action.id == "org.blueman.pppd.pppconnect") &&
        subject.isInGroup("sudo")) {

        return polkit.Result.YES;
    }
});
{{< /highlight >}}

If after every login bluetooth is auto-powered ON; this is due to Blueman’s _Power Manager_ plugin.  [Fix][Bluetooth No Auto-ON]: right-click Blueman applet → Plugins → PowerManager → Configuration; uncheck _Auto Power-on_.
<!-- TODO: update this in Arch Wiki - https://wiki.archlinux.org/index.php/Bluetooth#Auto_power-on_after_boot -->

The default Obex push directory is set to `~/.cache/obex`, named `Root`; change it to your convenience in _Local services_ -> _Transfer_.

[Blueman_permissions]: https://wiki.archlinux.org/index.php/Blueman#Permissions
[Bluetooth No Auto-ON]: https://www.linux.com/forums/networking/solved-bluez-543-have-bluetooth-disabled-boot

# Laptop Power Saving

`laptop-mode-tools` seems to be an important package for a laptop’s power saving.  Additionally you need `hdparm` and `cpupower`

{{< highlight basic >}}
yay -S --needed laptop-mode-tools hdparm cpupower
{{< /highlight >}}

Start with `sudo systemctl enable laptop-mode.service`.  Then set the following parameters in respective files

{{< highlight cfg >}}
# /etc/laptop-mode/conf.d/intel-sata-powermgmt.conf
CONTROL_INTEL_SATA_POWER="1"

# /etc/laptop-mode/conf.d/lcd-brightness.conf
BRIGHTNESS_OUTPUT="/sys/class/backlight/intel_backlight/brightness"
CONTROL_BRIGHTNESS=1

# /etc/laptop-mode/laptop-mode.conf
BATT_HD_POWERMGMT=200
LM_AC_HD_POWERMGMT=240
{{< /highlight >}}

Verify if last setting is in action with `hdparm -B /dev/sda`.

# Mount Removable Drives

If you’re dual booting, mounting NTFS partitions/disks is common; and then there’re USB drives too.  Though `/etc/fstab` is the traditional route for NTFS mounting, I don’t like this approach since this assumes you always need these drives.  Also you need empty directories ready to use as mount points for these drives.  It’d be good to do this automatically, on demand.

Enter [`udisks2`][udisks2] -- [the most-recommended approach][udisk recommendation]; many desktop environments seem to use it too.  This approach is intelligent compared to `fstab`’s plain hard-coding; respects per-user or group permissions; mount points are user-based too; having a [common mount point][] is also possible.  Install `udisks2` for conveniently mounting NTFS partitions and removable (USB) disks from the command-line or in [Thunar][].

{{< highlight basic >}}
yay -S --needed udisks2
{{< /highlight >}}

To allow users of group `sudo` to mount NTFS drives (with write permissions) without asking for `root` password, [enable the group in _polkit_][polkit sudo mount] under `/etc/polkit-1/rules.d/50-udisks.rules`

{{< highlight cfg >}}
polkit.addRule(function(action, subject) {
  var YES = polkit.Result.YES;
  var permission = {
    // only required for udisks2:
    "org.freedesktop.udisks2.filesystem-mount": YES,
    "org.freedesktop.udisks2.filesystem-mount-system": YES,
    "org.freedesktop.udisks2.encrypted-unlock": YES,
    "org.freedesktop.udisks2.eject-media": YES,
    "org.freedesktop.udisks2.power-off-drive": YES,
    // required for udisks2 if using udiskie from another seat (e.g. systemd):
    "org.freedesktop.udisks2.filesystem-mount-other-seat": YES,
    "org.freedesktop.udisks2.encrypted-unlock-other-seat": YES,
    "org.freedesktop.udisks2.eject-media-other-seat": YES,
    "org.freedesktop.udisks2.power-off-drive-other-seat": YES
  };
  if (subject.isInGroup("sudo")) {
    return permission[action.id];
  }
});
{{< /highlight >}}

From now on, you should see NTFS and USB disk partitions in Thunar’s sidebar.  On click mount and unmount will work.

[Thunar]: https://wiki.archlinux.org/index.php/Bluetooth#Auto_power-on_after_boot

## Mount/unmount from command-line

Un/mounting from the comfort of your command-line is even better.  Since you’d know your NTFS drives’ `/dev`, so you could

{{< highlight basic >}}
udisksctl mount --block-device /dev/sda3                         # NTFS disks
udisksctl unmount --block-device /dev/sda3
{{< /highlight >}}

I’ve a convenient Lua script to mount and unmount an array of NTFS partitions with a single command

{{< highlight basic >}}
#! /bin/env lua

local cmd = (#arg > 0) and (arg[1] == "u") and "unmount" or "mount"

os.execute("udisksctl " .. cmd .. " --block-device /dev/disk/by-label/Windows")
os.execute("udisksctl " .. cmd .. " --block-device /dev/disk/by-label/Data")
os.execute("udisksctl " .. cmd .. " --block-device /dev/disk/by-label/Media")
os.execute("udisksctl " .. cmd .. " --block-device /dev/disk/by-label/Backup")
{{< /highlight >}}

Since you’d be having `udev` running, it should [populate][by-label populator] `/dev/disk/by-label` which might be useful for USB drives.

{{< highlight basic >}}
udisksctl mount --block-device /dev/disk/by-label/CAPSULE
udisksctl unmount --block-device /dev/disk/by-label/CAPSULE
udisksctl power-off --block-device /dev/disk/by-label/CAPSULE    # power-off before unplug
{{< /highlight >}}

A recommended GUI, if needed, is [udiskie](https://github.com/coldfix/udiskie).

[by-label populator]: https://unix.stackexchange.com/q/56291/30580
[udisk recommendation]: https://wiki.archlinux.org/index.php/USB_storage_devices#Auto-mounting_with_udisks
[udisks2]: https://wiki.archlinux.org/index.php/Udisks
[common mount point]: https://wiki.archlinux.org/index.php/Udisks#Mount_to_/media_(udisks2)
[polkit sudo mount]: https://unix.stackexchange.com/a/207667

# Package Management, yay!

Many essential packages live in [AUR][], the unofficial Arch repository; just the official repositories won’t cut it.  The default package manager client `pacman` works only with official repos.  Many famous [AUR helpers][] and [pacman wrappers][] have been written hence.  The former deals only with AUR packages, while letting `pacman` work with the official repo packages; the latter is more wholesome.  Pacman wrappers take care of both official and AUR repos; effectively letting the user deal with both transparently e.g. `yay -Syu` updates all packages irrespective of their repo.

[Yay][] --- the pacman wrapper written in [Go][] --- simple interface, feature-rich and an active project.  I recommend it over the many, now-defunct, pacman wrappers out there.  Once installed, Yay should take care of your AUR (and official repo) package needs.  However, to get Yay itself, you need to do manual AUR package installation.  To make packages from AUR you need the `base-devel` package group and super user permissions; you can’t `sudo` before getting added to the group, so

{{< highlight basic >}}
pacman -S --needed base-devel
su
visudo                      # uncomment sudo group
groupadd sudo               # if not already existing
gpasswd -a sundaram sudo
{{< /highlight >}}

Now install Yay from AUR

{{< highlight basic >}}
git clone https://aur.archlinux.org/yay.git
cd yay/
makepkg -si    # this needs sudo to install built pacakge
{{< /highlight >}}

I [found][Package Mapping] a couple of useful tricks as (my machine’s) admin:

* Map an existing file back to its (installed) package: `pacman -Qo FILE-PATH`
{{< highlight basic >}}
> pacman -Qo mandelbrot
/usr/bin/mandelbrot is owned by mesa-demos 8.4.0-1
{{< /highlight >}}
* Map any command/file name (not necessarily existing) to its (to-be-installed) package: `pkgfile FILE`
{{< highlight basic >}}
> pkgfile hexl
extra/emacs
community/emacs-nox
{{< /highlight >}}
* List installed packages with [fzf][]; a panel shows highlighted package’s details
{{< highlight basic >}}
> yay -Qq | fzf --preview 'yay -Qil {}' --layout=reverse --bind 'enter:execute(yay -Qil {} | less)'
{{< /highlight >}}


[AUR]: https://aur.archlinux.org/
[pacman wrappers]: https://wiki.archlinux.org/index.php/AUR_helpers#Pacman_wrappers
[AUR helpers]: https://wiki.archlinux.org/index.php/AUR_helpers
[Yay]: https://github.com/Jguer/yay
[Go]: https://golang.org/
[Package Mapping]: https://bbs.archlinux.org/viewtopic.phpid=90635
[fzf]: https://github.com/junegunn/fzf

# Fonts

I’m a [Tamilian][tamils] and a programmer.  Since most of my consumption is textual data, I’ve strong tastes in fonts.  First for greater unicode coverage I followed Emacs’ [unicode-fonts][] recommendation of at least installing

* DejaVu Sans
* Symbola
* Quivira
* Noto TTFs

{{< highlight basic >}}
yay -S --needed ttf-dejavu  ttf-symbola noto-fonts
{{< /highlight >}}

[Quivira][] wasn’t in the Arch repos; installed manually by copying into `${HOME}/.local/share/fonts`.

Noto has excellent coverage across languages.  With these my missing-glyph agonies were gone.

## Tamil

To have Tamil rendered in Firefox (both content and UI like menu, etc.)

{{< highlight basic >}}
yay -S --needed ttf-tamil firefox-i18n-ta
{{< /highlight >}}

Looks beautiful 😇 அருமை அருமை!

## Monospace

I seem to be partial to fonts with curves.  I’m a fan of [Mononoki][] and Ubuntu Mono.

{{< highlight basic >}}
yay -S --needed ttf-ubuntu-font-family ttf-mononoki
{{< /highlight >}}

This starts showing up inside the browser --- for code snippets --- too!

[Nerd Fonts][] is a project that lets you impregnate your favourite font with glyphs from icon packages like [Font Awesome][], [Devicons][], ….  It’s useful if you’re used to using these special icons.  Most popular fonts don’t need anything manual -- the legwork is already done; just download!  I removed `ttf-mononoki` and installed _mononoki Nerd Font_ manually.

## Xfce4 Settings

To see available fonts, install `fontconfig` and do `fc-list`.

Under _Appearance_ set

* **Default Font**: Noto Sans 12
* **Default Monospace Font**: mononoki 13

However, for monospace this wasn’t enough 🤦.  For instance, Terminal and Mousepad ignores it.  I don’t want to override for each app manually either.

{{< highlight basic >}}
gsettings set org.gnome.desktop.interface monospace-font-name 'mononoki 13'
{{< /highlight >}}

does it!  This also fixes the inline code face used by Emacs’ `markdown-mode` 😮

Another option is to customize font setting per user by fixing `{HOME}/.config/fontconfig/fonts.conf`.  I did this too for good measure.

{{< highlight basic >}}
<match target="pattern">
  <test name="family" qual="any">
    <string>monospace</string>
  </test>
  <edit binding="strong" mode="prepend" name="family">
    <string>mononoki</string>
  </edit>
</match>
{{< /highlight >}}

I got both of these from [Unix.StackExchange][monospace-everywhere].

[tamils]: https://en.wikipedia.org/wiki/Tamils
[unicode-fonts]: https://github.com/rolandwalker/unicode-fonts
[Quivira]: http://www.quivira-font.com/files/Quivira.ttf
[Mononoki]: https://madmalik.github.io/mononoki/
[monospace-everywhere]: https://unix.stackexchange.com/questions/106070/changing-monospace-fonts-system-wide
[Nerd Fonts]: https://nerdfonts.com/
[Font Awesome]: https://fontawesome.com/
[Devicons]: http://vorillaz.github.io/devicons

# User land

Miscellaneous user land customizations and tune-ups:

* Archive Manager
    - `yay -S --needed p7zip unzip unrar`
    - `xarchiver` GUI integrates well with Thunar
* Lock screen
    - `xlockmore` works fine, just be aware that switching to TTY (with Ctrl + Alt + 2, ...) is still possible with it
    - Integrates seamlessly with `xflock4` a script (`/usr/bin/xflock4`) which tries different lockers
    - Other lockers have issues
        - `gnome-screensaver`, `sflock` -- didn’t work
        - `physlock` -- only `root` can unlock
        - `light-lock` -- needs a display manager
* Screenshots
    - `xfce4-screenshooter`; had to be hooked to <kbd>PrintScr</kbd> through `xfce4-keyboard-settings` under the _Application Shortcuts_ tab
* Preferred Applications
    - `xdg-utils` (a dependency of packages like `mpv`, `blender`, etc.)
        - `xdg-open` opens file with preferred application from terminal
    - Another option: `perl-file-mimeinfo`
    - Xfce4 has _MIME Type Editor_ whose settings both MC and Thunar respect
        - When something unassociated is opened in Thunar, what you choose gets updated here only
* Xfce4 Terminal Colour Scheme
    - Pick-up from [iTerm2 Color Schemes][]
    - Copy to `${HOME}/.local/share/xfce4/terminal/colorschemes`
* Thunar + Emacs
    - _Edit with Emacs_ from Thunar by making a `emacs.desktop`; see [Emacs.StackExchange][emacs.desktop]
* Cloud Sync
    - [rclone][] syncs to remote drives; all popular cloud storage services are supported.
    - You also make it a [VFS][cloud-vfs]!
* Books
    - PDF: [Evince][]
    - CHM: [xCHM][]
    - ePub: [Bookworm][]
* Images
    - Command line: [feh]
    - GUI: [Ristretto][], part of `xfce4-goodies`
* Media
    - Per-file A/V playback: [mpv][]
    - Console music player/library: [Music on Console][]
    - Conversion: [ffmpeg][]
    - Encoding: [Handbrake][]

[iTerm2 Color Schemes]: https://iterm2colorschemes.com/
[emacs.desktop]: https://emacs.stackexchange.com/q/14055/4106
[cloud-vfs]: https://www.everything-linux-101.com/blog/mount-onedrive-in-linux/
[feh]: https://feh.finalrewind.org/
[Bookworm]: https://babluboy.github.io/bookworm/
[xCHM]: https://github.com/rzvncj/xCHM
[Evince]: https://wiki.gnome.org/Apps/Evince
[rclone]: https://rclone.org/
[Ristretto]: https://docs.xfce.org/apps/ristretto/start
[mpv]: https://mpv.io/
[Music on Console]: https://moc.daper.net/
[ffmpeg]: https://ffmpeg.org/
[Handbrake]: https://handbrake.fr/

# Issues

1. Log out and in; mouse cursor is frozen!
2. Reverse scrolling in Xfce4 Terminal.

# Epilogue

Phew!  What a long post!  Despite those petty issues, it’s an _amazingly productive and stable_ setup.

> I find excuses to use this environment; smooth and pleasant!

I hope this helps someone trying to figure out stuff in Arch or Linux in general. If you’ve tips or suggestions to share, you’re most welcome!

Thanks to all those who report issues, document fixes, share and help unknown people in open forums like StackExchange, GitHub, Reddit, ….

Hat tip to all developers/creators who’ve worked to create this synergy ☯.

# References

1. Arch Linux Installation Guide
2. [Installing Arch Linux with LVM](https://gilyes.com/Installing-Arch-Linux-with-LVM/) by George Ilyes
3. [Multi HDD/SSD Partitioning Scheme](https://wiki.debian.org/Multi HDD/SSD Partition Scheme)
4. [GRUB](https://wiki.archlinux.org/index.php/GRUB#Check_for_an_EFI_System_Partition) on Arch Linux Wiki
5. [Dual-boot with Windows](https://wiki.archlinux.org/index.php/Dual_boot_with_Windows#UEFI_systems)
6. [Pick A Suitable Desktop Environment For Arch Linux](https://www.2daygeek.com/install-xfce-mate-kde-gnome-cinnamon-lxqt-lxde-budgie-deepin-enlightenment-desktop-environment-on-arch-linux/)
