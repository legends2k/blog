+++
title = "Arch Linux Configuration"
description = "Display, Audio, Bluetooth and tools"
date = "2018-05-27T17:00:46-07:00"
publishDate = "2019-04-26T11:31:00+05:30"
tags = ["tech", "tools", "linux"]
toc = true
+++

See [_Arch Linux Installation_](/note/arch_install) for installation notes.

# Network

In the newly installed Arch you might notice that there's no network connectivity.  [Fix it with `iwctl`](/note/arch_install).
[iwd][] seems to be the simplest and fastest for wireless network on Linux.  It’s self-sufficient having a built-in DHCP client.  Manually fixing `/etc/resolv.conf` with some standard DNS addresses should get you online.  However, I prefer `systemd-resolved` for DNS and `systemd-networkd` for ethernet connections.  I referred [insanity.industries’ excellent guide][nw-setup] to setup all of these.  My final `/etc/iwd/main.conf`:

{{< highlight cfg >}}
[General]
EnableNetworkConfiguration=true
AddressRandomization=once

[Scan]
DisablePeriodicScan=true

[Network]
NameResolvingService=systemd
{{< /highlight >}}

and `/etc/systemd/resolved.conf`:

{{< highlight cfg >}}
[Resolve]
FallbackDNS=1.1.1.1 1.0.0.1
{{< /highlight >}}

This way an access point’s DNS is preferred over the fallback Cloudfare’s.

[iwd]: https://wiki.archlinux.org/title/iwd
[nw-setup]: https://insanity.industries/post/simple-networking/

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
pacman -S --needed xfce4 xfce4-goodies lxdm
{{< /highlight >}}

Set `graphical.target` as default.  Start and enable (for future boots) the display manager

{{< highlight basic >}}
systemctl set-default graphical.target
systemctl enable --now lxdm.service
{{< /highlight >}}

For the first run, make sure to set the _Session_ and _Locale_; not doing so led to a login screen loop.

If LXDM doesn’t show your full name, it isn’t set in `/etc/passwd`.  Set it with `chfn -f 'Full Name' login_id`.

For natural scrolling with laptop’s touchpad enable _Reverse scroll direction_ under _Mouse and Touchpad_ settings.

# Display

I got infinite waits every time I tried `poweroff -n` due to Nouveau drivers for Nvidia; for this reason I disabled `lxdm.service` and operated from the terminal.  To list the graphics devices you’ve, do

{{< highlight basic >}}
lspci -k | grep -A 2 -E "(VGA|3D)"
{{< /highlight >}}

## Intel

With Skylake and its successors (Kabylake, …) we can enable `i915` module for [early KMS start][] in `/etc/mkinitcpio.conf` and run `mkinitcpio -p linux`

{{< highlight cfg >}}
# MODULES
# …
MODULES=(i915)
{{< /highlight >}}

That’s it!  Things work out of the box.  You can check that GuC, HuC and FBC are enabled by default

{{< highlight basic >}}
# options with defaults
modinfo -p i915
# currently enabled options
systool -m i915 -av
{{< /highlight >}}

[early KMS start]: https://wiki.archlinux.org/index.php/Kernel_mode_setting#Early_KMS_start

## Nvidia

Using the discrete GPU for heavier workloads and powering it off (as integrated GPU does display stuff) would be ideal.  Official (nvidia) method is to go with `prime-run`.  Following [PRIME - ArchWiki][] did most of the trick but power management isn’t there yet.

{{< highlight basic >}}
pacman -S nvidia nvidia-utils nvidia-prime
systemctl enable --now nvidia-persistanced
{{< /highlight >}}

`prime-run` a process to choose discrete GPU; the default would be integrated:

{{< highlight basic >}}
# should display integrated GPU
glxinfo -B
# should display discrete GPU
prime-run glxinfo -B
# useful (informative)
nvidia-smi
# information on GPU
cat /proc/driver/nvidia/gpus/0000\:01\:00.0/information
{{< /highlight >}}

[Dynamic power management][nv-dyn-power] is only for Turing+ architectures.  Installing `nvidia-prime-rtd3pm` at least sets `/sys/bus/pci/devices/0000:01:00.0/power/control` to `auto` (better than `on`).  `cat /sys/bus/pci/devices/0000:01:00.0/power_state` always shows `D0`, `cat /proc/driver/nvidia/gpus/0000\:01\:00.0/power` shows `Disabled by default`.  What more, `nvidia-smi` isn’t able to show power stats as it’s unsupported for this GPU 🤦

[nv-dyn-power]: https://us.download.nvidia.com/XFree86/Linux-x86_64/525.89.02/README/dynamicpowermanagement.html
[prime - archwiki]: https://wiki.archlinux.org/title/PRIME

# USB Port Speed

If your machine has both USB 2 and 3 ports and boot logs show this warning

{{< highlight basic >}}
kernel: usb: port power management may be unreliable
{{< /highlight >}}

there’s a good chance of a USB 3+ device (5000 Mbps) running with USB 2 (480 Mbps) speed; verify with `lsusb -tvv`.  Refer [UnixSE][] and [ArchLinux forums][arch-usb3-issue] for details.  The problem maps to inability of the kernel to determine a port’s peer and set correct power.  If that’s the case, find the port (`dmesg | grep -i usb`) and manually suspend it; [refer][baeldung-usb-power]

{{< highlight basic >}}
echo "0" > "/sys/bus/usb/devices/1-1:1.0/power/autosuspend_delay_ms"
echo "auto" > "/sys/bus/usb/devices/1-1:1.0/power/control"
{{< /highlight >}}

[unixse]: https://unix.stackexchange.com/q/323692/30580
[arch-usb3-issue]: https://bbs.archlinux.org/viewtopic.php?id=219465
[baeldung-usb-power]: https://www.baeldung.com/linux/control-usb-power-supply

# Audio

Install `pipewire-alsa`, `pipewire-pulse`, `pavucontrol` and `xfce4-pulseaudio-plugin`; no fiddling was needed to get audio working.  Pipewire + Wireplumber = 🎵💘!

After installing `pipewire-alsa`, my bluetooth headphones with handsfree microphone, earlier showing up only as Audio-Out device, exposes UI to pick between A2DP Sink and HSP/HFP profiles in _Bluetooth Manager_!  Switching to the latter profile shows a new Audio-In device under `pavucontrol` :)

# Bluetooth

Install `bluez`, `bluez-utils` and `blueman` -- for a decent GTK+ bluetooth manager with an applet.  Verify if bluetooth hardware is not hard blocked but is soft blocked.

{{< highlight basic >}}
rfkill list highlight

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
usermod -aG lp $USER
{{< /highlight >}}

Enable and start `bluetooth.service`.

{{< highlight basic >}}
systemctl enable --now bluetooth.service
{{< /highlight >}}

Though this is enough, at every login, there’d be an annoying prompt for the root password, to enable bluebooth.  Add this to `/etc/polkit-1/rules.d/51-blueman.rules` to not prompt for users in the `wheel` group as [detailed in the Wiki][Blueman_permissions]:

{{< highlight cfg >}}
/* Allow users in wheel group to use blueman feature requiring root without authentication */
polkit.addRule(function(action, subject) {
    if ((action.id == "org.blueman.network.setup" ||
         action.id == "org.blueman.dhcp.client" ||
         action.id == "org.blueman.rfkill.setstate" ||
         action.id == "org.blueman.pppd.pppconnect") &&
        subject.isInGroup("wheel")) {

        return polkit.Result.YES;
    }
});
{{< /highlight >}}

If after every login bluetooth is auto-powered ON; this is due to Blueman’s _Power Manager_ plugin.  [Fix][Bluetooth No Auto-ON]: right-click Blueman applet → Plugins → PowerManager → Configuration; uncheck _Auto Power-on_.
<!-- TODO: update this in Arch Wiki - https://wiki.archlinux.org/index.php/Bluetooth#Auto_power-on_after_boot -->

The default Obex push directory is set to `~/.cache/obex`, named `Root`; change it to your convenience in _Local services_ -> _Transfer_.

To show battery charge in bluetooth devices like headphones (as passive desktop notifications too) enable D-Bus experimental features in `/etc/bluetooth/main.conf`:

{{< highlight cfg >}}
Experimental = true
{{< /highlight >}}

[Blueman_permissions]: https://wiki.archlinux.org/index.php/Blueman#Permissions
[Bluetooth No Auto-ON]: https://www.linux.com/forums/networking/solved-bluez-543-have-bluetooth-disabled-boot

# Laptop Power Saving

`laptop-mode-tools` seems to be an important package for a laptop’s power saving.  Additionally you need `hdparm` and `cpupower`

{{< highlight basic >}}
yay -S --needed laptop-mode-tools hdparm cpupower
{{< /highlight >}}

Start with `systemctl enable laptop-mode.service`.  Then set the following parameters in respective files

{{< highlight cfg >}}
# /etc/laptop-mode/conf.d/intel-sata-powermgmt.conf
CONTROL_INTEL_SATA_POWER="1"

# /etc/laptop-mode/conf.d/lcd-brightness.conf
# Follow comments to find brightness <value>s
CONTROL_BRIGHTNESS=1
BATT_BRIGHTNESS_COMMAND="echo 1500"
LM_AC_BRIGHTNESS_COMMAND="echo 2250"
NOLM_AC_BRIGHTNESS_COMMAND="echo 2250"
BRIGHTNESS_OUTPUT="/sys/class/backlight/intel_backlight/brightness"

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

To allow users of group `wheel` to mount NTFS drives (with write permissions) without asking for `root` password, [enable the group in _polkit_][polkit wheel mount] under `/etc/polkit-1/rules.d/50-udisks.rules`

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
  if (subject.isInGroup("wheel")) {
    return permission[action.id];
  }
});
{{< /highlight >}}

From now on, you should see NTFS and USB disk partitions in Thunar’s sidebar.  One click mount and unmount will work.

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

A recommended GUI, if needed, is [udiskie](https://github.com/coldfix/udiskie).  Thunar auto-integrates well though.

[by-label populator]: https://unix.stackexchange.com/q/56291/30580
[udisk recommendation]: https://wiki.archlinux.org/index.php/USB_storage_devices#Auto-mounting_with_udisks
[udisks2]: https://wiki.archlinux.org/index.php/Udisks
[common mount point]: https://wiki.archlinux.org/index.php/Udisks#Mount_to_/media_(udisks2)
[polkit wheel mount]: https://unix.stackexchange.com/a/207667

# Package Management, yay!

Many essential packages live in [AUR][], the unofficial Arch repository; just the official repositories won’t cut it.  The default package manager client `pacman` works only with official repos.  Many famous [AUR helpers][] and [pacman wrappers][] have been written hence.  The former deals only with AUR packages, while letting `pacman` work with the official repo packages; the latter is more wholesome.  Pacman wrappers take care of both official and AUR repos; effectively letting the user deal with both transparently e.g. `yay -Syu` updates all packages irrespective of their repo.

[Yay][] --- the pacman wrapper written in [Go][] --- simple interface, feature-rich and an active project.  I recommend it over the many, now-defunct, pacman wrappers out there.  Once installed, Yay should take care of your AUR (and official repo) package needs.  However, to get Yay itself, you need to do manual AUR package installation.  To make packages from AUR you need the `base-devel` package group and super user permissions; you can’t `sudo` before getting added to sudo/wheel group

{{< highlight basic >}}
pacman -S --needed base-devel
su
EDITOR=/usr/bin/nano visudo    # allow wheel group members to sudo
groupadd wheel                 # if not already existing
usermod -aG wheel $USER
sudo -e /etc/makepkg.conf      # disable *-debug builds: OPTIONS=(… !debug)
{{< /highlight >}}

Install pre-built Yay from AUR

{{< highlight basic >}}
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin/
makepkg -si    # this needs sudo to install built package
{{< /highlight >}}

Yay respects `/etc/pacman.conf` settings; enable `Color` and `ParallelDownloads` -- very useful!  I [found][Package Mapping] a few useful tricks as (my machine’s) admin:

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

Irrespective of the wrapper used, pacman maintains `/var/log/pacman.log` since day 1 of its usage; very useful to know package management history.

## Pac Cache Management

Periodically `/var` gets full due to cached downloaded packages.  Clear it manually using the `paccache` script

{{< highlight basic >}}
paccache -rk 2    # keep last 2 versions of each package
{{< /highlight >}}

[Automate cache clearance][autorun-paccache] with a pacman hook (refer `man alpm-hooks`) run post install:

{{< highlight cfg >}}
cat > /etc/pacman.d/hooks/clean_pac_cache.hook
[Trigger]
Operation = Upgrade
Operation = Install
Operation = Remove
Type = Package
Target = *

[Action]
Description = Cleaning pacman cache...
When = PostTransaction
Exec = /usr/bin/paccache -rk 1
{{< /highlight >}}

If you’ve issues with keyring or keys, [reinitialize keyring][reinit-keyring].

[AUR]: https://aur.archlinux.org/
[pacman wrappers]: https://wiki.archlinux.org/index.php/AUR_helpers#Pacman_wrappers
[AUR helpers]: https://wiki.archlinux.org/index.php/AUR_helpers
[Yay]: https://github.com/Jguer/yay
[Go]: https://golang.org/
[Package Mapping]: https://bbs.archlinux.org/viewtopic.php?id=90635
[fzf]: https://github.com/junegunn/fzf
[autorun-paccache]: https://ostechnix.com/recommended-way-clean-package-cache-arch-linux/
[reinit-keyring]: https://stackoverflow.com/a/73001822/183120

# Fonts

I’m a [Tamilian][Tamils] and a programmer.  Since most of my consumption is textual data, I’ve strong tastes in fonts.  First for greater unicode coverage I followed Emacs’ [unicode-fonts][] recommendation of at least installing

* DejaVu Sans
* Symbola
* Quivira
* Noto TTFs

{{< highlight basic >}}
yay -S --needed ttf-dejavu  ttf-symbola quivira noto-fonts noto-fonts-emoji
{{< /highlight >}}

Fonts can be installed manually by copying into `${HOME}/.local/share/fonts`.

Noto has excellent coverage across languages.  [`noto-fonts-emoji` is the Emacs-recommended font for Emojis][emacs-noto-emoji]; xfce4-terminal shows them too 🤗.  With these my missing-glyph agonies were gone.

## Tamil

To have Tamil rendered in Firefox (both content and UI like menu, etc.)

{{< highlight basic >}}
yay -S --needed ttf-tamil firefox-i18n-ta
{{< /highlight >}}

Looks beautiful 😇 அருமை அருமை!

## Monospace

I seem to be partial to fonts with curves.  I’m a fan of [Mononoki][] and Ubuntu Mono.

{{< highlight basic >}}
yay -S --needed ttf-ubuntu-font-family ttf-mononoki-nerd
{{< /highlight >}}

This starts showing up inside the browser --- for code snippets --- too!

[Nerd Fonts][] is a project that lets you impregnate your favourite font with glyphs from icon packages like [Font Awesome][], [Devicons][], ….  It’s useful if you’re used to using these special icons.  Most popular fonts don’t need anything manual -- the legwork is already done; just download!

## Xfce4 Settings

To see available fonts, run `fc-list` (from `fontconfig` package)

Under _Appearance_ set

* **Default Font**: Noto Sans Regular 12
* **Default Monospace Font**: Mononoki Nerd Font Mono Regular 13

If setting system monospace wasn’t enough

{{< highlight basic >}}
gsettings set org.gnome.desktop.interface monospace-font-name 'mononoki 13'
{{< /highlight >}}

does it!  This also fixes the inline code face used by Emacs’ `markdown-mode` 😮

Another option is to customize font setting per user by fixing `{HOME}/.config/fontconfig/fonts.conf`.  I did this too for good measure.

{{< highlight xml >}}
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

[Tamils]: https://en.wikipedia.org/wiki/Tamils
[unicode-fonts]: https://github.com/rolandwalker/unicode-fonts
[emacs-noto-emoji]: https://www.masteringemacs.org/article/whats-new-in-emacs-28-1
[Mononoki]: https://madmalik.github.io/mononoki/
[monospace-everywhere]: https://unix.stackexchange.com/questions/106070/changing-monospace-fonts-system-wide
[Nerd Fonts]: https://nerdfonts.com/
[Font Awesome]: https://fontawesome.com/
[Devicons]: http://vorillaz.github.io/devicons

# Multi-monitor Setup

Targus DisplayLink DOCK180USZ setup was easy; followed first few steps of [DisplayLink - ArchWiki][] and fiddled with _Display_ settings.

{{< highlight basic >}}
yay -S evdi displaylink linux-headers
systemctl enable --now displaylink.service
cat > /etc/X11/xorg.conf.d/20-evdi.conf
Section "OutputClass"
	Identifier "DisplayLink"
	MatchDriver "evdi"
	Driver "modesetting"
	Option "AccelMethod" "none"
EndSection
{{< /highlight >}}

[displaylink - archwiki]: https://wiki.archlinux.org/title/DisplayLink

# Memory Card Write Access

Add to `storage` group to get read-write, instead of read-only, access to SD cards.

{{< highlight basic >}}
usermod -aG storage $USER
{{< /highlight >}}

# User land

Miscellaneous user land customizations and tune-ups:

* Manuals
    - `yay -S --needed texinfo`
    - `for f in /usr/share/info/*;  do install-info ${f} /usr/share/info/dir 2>/dev/null; done`
        + Unneeded ideally but texinfo’s `post_install()` doesn’t do it
* plocate, findutils, binutils, util-linux, pax-utils (`lddtree`), pkgfile, man-db
    - `systemctl enable --now plocate-updatedb.timer pkgfile-update.timer man-db.timer`
    - Check timers: `systemctl list-timers`
* Archive Manager
    - `yay -S --needed 7zip unzip unrar`
    - `xarchiver` GUI integrates well with Thunar
* `light-locker` is pretty good as a lock screen
* Screenshots
    - `xfce4-screenshooter`; had to be hooked to <kbd>PrintScr</kbd> through `xfce4-keyboard-settings` under the _Application Shortcuts_ tab
* Preferred Applications
    - `xdg-utils` (a dependency of packages like `mpv`, `blender`, etc.)
        + `xdg-open` opens file with preferred application from terminal
    - Another option: `perl-file-mimeinfo`
    - Xfce4 has _MIME Type Editor_ whose settings both MC and Thunar respect
        + When something unassociated is opened in Thunar, what you choose gets updated here only
* Xfce4 Terminal Colour Scheme
    - Pick-up from [iTerm2 Color Schemes][]
    - Copy to `${HOME}/.local/share/xfce4/terminal/colorschemes`
* Thunar + Emacs
    - _Edit with Emacs_ from Thunar by making a `emacs.desktop`; thanks [Emacs.StackExchange][emacs.desktop]
* Cloud Sync
    - [rclone][] syncs to remote drives; all popular cloud storage services are supported.
    - You can also make it a [VFS][cloud-vfs]!
* Books
    - PDF: [Evince][]
    - CHM: [xCHM][]
    - ePub: [Foliate][]
* Images
    - [ImageMagick][], the swiss-army knife of image formats
    - [geeqie][] (GUI) -- versatile; supports [many formats][geeqie-formats] including camera RAW and vector
* Media
    - Per-file A/V playback: [mpv][]
    - Console music player/library: [Music on Console][]
    - Conversion: [ffmpeg][]
    - Encoding: [Handbrake][]

[iTerm2 Color Schemes]: https://iterm2colorschemes.com/
[emacs.desktop]: https://emacs.stackexchange.com/q/14055/4106
[cloud-vfs]: https://www.everything-linux-101.com/blog/mount-onedrive-in-linux/
[geeqie]: http://www.geeqie.org/
[geeqie-formats]: https://github.com/BestImageViewer/geeqie#features
[Foliate]: https://johnfactotum.github.io/foliate/
[xCHM]: https://github.com/rzvncj/xCHM
[Evince]: https://wiki.gnome.org/Apps/Evince
[rclone]: https://rclone.org/
[Ristretto]: https://docs.xfce.org/apps/ristretto/start
[mpv]: https://mpv.io/
[Music on Console]: https://moc.daper.net/
[ffmpeg]: https://ffmpeg.org/
[Handbrake]: https://handbrake.fr/
[imagemagick]: https://imagemagick.org/

# Issues

Check for kernel issues during boot up and down with

{{< highlight bash >}}
journalctl --list-boots
# Check current boot’s logs, use ‘-b -1’ for previous, etc.
journalctl -b [-0]
{{< /highlight >}}

1. Log out and in; mouse cursor is frozen!
    - Fix: `sudo modprobe -r psmouse && modprobe psmouse`

# Epilogue

Phew!  What a long post!  Despite those petty issues, it’s an _amazingly productive and stable_ setup.

> I find excuses to use this environment; smooth and pleasant!

I hope this helps someone trying to figure out stuff in Arch or Linux in general. If you’ve tips or suggestions to share, you’re most welcome!

Thanks to all those who report issues, document fixes, share and help unknown people in open forums like StackExchange, GitHub, Reddit, ….

Hat tip to all developers/creators who’ve worked to create this synergy ☯.

# References

1. [Installation guide -- ArchWiki](https://wiki.archlinux.org/index.php/Installation_guide)
2. [Installing Arch Linux with LVM](https://gilyes.com/Installing-Arch-Linux-with-LVM/) by George Ilyes
3. [Multi HDD/SSD Partitioning Scheme](https://wiki.debian.org/Multi%20HDD/SSD%20Partition%20Scheme)
4. [GRUB](https://wiki.archlinux.org/index.php/GRUB#Check_for_an_EFI_System_Partition) on Arch Linux Wiki
5. [Dual-boot with Windows](https://wiki.archlinux.org/index.php/Dual_boot_with_Windows#UEFI_systems)
6. [Pick A Suitable Desktop Environment For Arch Linux](https://www.2daygeek.com/install-xfce-mate-kde-gnome-cinnamon-lxqt-lxde-budgie-deepin-enlightenment-desktop-environment-on-arch-linux/)

# See Also

1. [Linux Desktop Setup](https://hookrace.net/blog/linux-desktop-setup/)
