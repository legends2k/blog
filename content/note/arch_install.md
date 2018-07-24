+++
title = "Arch Linux Installation"
description = "coexisting with Windows on a dual GPU laptop"
date = "2018-05-27T17:00:46-07:00"
tags = ["tech", "tools", "writing"]
draft = true
+++

title = "Hello, Hugo!"
description = "static site generators and how they can help"
date = "2016-07-11T17:00:46+05:30"
tags = ["tech", "tools", "writing"]
mathjax = true


I've a _Lenovo Legion Y520-15IKBN_ laptop which has _Intel HD Graphics 630_ for simpler tasks and _Nvidia GeForce 1050 Ti Mobile_ for hungrier (3D) work.  It has an SSD and a traditional spinning disk.  It uses UEFI + GPT as opposed to the classic BIOS + MBR combo.  I'm a long time user of Xubuntu and I wanted to try Arch Linux with Xfce.  I want to document my experience in getting this laptop up and running.

# Prelims

A few things upfront --- detailed later --- to decide if this guide is for you

1. **Secure boot** off: at least for (USB) installation.  Longer term, there're some hoops to jump through but you can enable it back on
2. **SSD vs non-SSD**: Use spinning disks for volatile data
3. **SATA vs RAID**: To even detect your M.2 PCIe SSD the storage controller is to be set to ACHI and not Intel RST
4. **LVM**: if you want the flexibility of changing partition sizes without data loss, post installation, even while mounted

Being a Emacs junkie one of the first things I do when I start using a new environment is remap my <kbd>CAPSLOCK</kbd> to <kbd>CONTROL</kbd>.

## Ctrl your CapsLk

[Map `CAPSLOCK` to `CTRL`](https://www.emacswiki.org/emacs/MovingTheCtrlKey#toc8) with `loadkeys` [just for the current session](https://wiki.archlinux.org/index.php/Keyboard_configuration_in_console)

```bash
dumpkeys | head -1 > keys.map`
echo 'keycode 58 = Control' >> keys.map
loadkeys keys.map
```

Once we’ve a desktop environment setup, this can be made permanent with [`.Xmodmap`](https://bitbucket.org/rmsundaram/tryouts/src/dev/Misc/config/.Xmodmap).

# Wireless Network

Without network access modern linux installations are an exercise in pain.  I didn't have an ethernet cable so I had to get wi-fi up:

1. If `ping` fails, disable _dhcpcd_ service with `systemctl stop dhcpcd@` (press <kbd>TAB</kbd>)
2. Check wireless interface creation with `iw dev`; it should list the interface name e.g. `wlp3s0`
    * If absent, `lspci -v` to ensure the wireless network controller drivers loaded right
    * `ip link show` shows the networking interfaces present
3. Check if wireless is not blocked with `rfkill list`
    * If hard-blocked, press hardware button to enable wireless hardware
    * If soft-blocked: `rfkill unblock wifi`
4. `cp /etc/netctl{/examples,}/wireless-wpa`
    * Just copy an example profile and tweak
    * Arch comes with _netctl_ (network manager) and _WPA supplicant_ for connection to WPA secured networks
    * Can't use _iw_ or _wireless\_tools_ if the network uses _WPA2 Personal_
5. Edit parameters in copied file as per network
    * Use `wpa_passphrase` to get the hex passphrase to put in netctl's profile
    * Make sure you run `wpa_passphrase` with the correct SSID

```
Description='A simple WPA encrypted wireless connection'
Interface=wlp3s0
Connection=wireless

Security=wpa
IP=dhcp

ESSID='InfoProbe'
Key=\72ac...
Hidden=yes
```

1. `netctl start wireless-wpa`
2. Ping should now work if all went well

In case `netctl` errors out, it might be because of two things:

1. A DHCP daemon is still running; do `systemctl stop ...` as in step 1
2. The interface is already up; [netctl needs it down](https://bbs.archlinux.org/viewtopic.php?id=162582): `ip link set wlp3s0 down`

# Partitions

## SSD

Check if the drive got detected `ls /dev/nv*`.  If not, the SSD is probably configured to use the RST (RAID) storage controller which Linux doesn't seem to detect.  It has to be AHCI (SATA) controller.

If the default Windows installation was done with RST selection, Windows 10 will not boot if this switch is flipped.  Recovering Windows without a reinstallation [is possible](http://triplescomputers.com/blog/uncategorized/solution-switch-windows-10-from-raidide-to-ahci-operation/).  Minimal _Safe Mode_ can be enabled/disabled without using `bcdedit` [through GUI](http://www.thewindowsclub.com/reboot-in-safe-mode-windows) too: `msconfig`.  [Tests show](https://www.reddit.com/r/Dell/comments/4gke4k/a_closer_look_at_ahci_vs_raid/) that using the older AHCI doesn't incur any significant performance hit over RAID.

## Scheme

SSDs are [not recommended](https://unix.stackexchange.com/a/89230) for volatile data, so `/var` and swap space should be allocated on spinning disk.  Rest like `/boot`, `/`, `/usr`, etc. can be left on the SSD: seen under `/dev/nvme0nXpY` not `/dev/sda`; `X` is the disk number, `Y` is the partition.

Partition with `cfdisk` and verify them with `lsblk` or `fdisk -l`.  Choose _Linux filesystem_ for _Partition Type_ for everything except swap and LVM.  Mind that these partitions are the ones created outside the LVM; if you're going to manage all your Linux partitions with LVM, just create one partition and set its type to _Linux LVM_ in `fdisk`.

My final scheme (sublist intelligible after reading the next section)

1. `/boot` → `/dev/nvme0n1p5` (512 MiB)
2. LVM → `/dev/nvme0n1p6` (54.5 GiB)
    1. `/` → `/dev/lvmg1/root` (40 GiB)
    2. `/home` → `/dev/lvmg1/home` (remaining)
3. `/var` → `dev/sda5` (4 GiB)
4. swap → `dev/sda6` (4 GiB)

## LVM

Using LVM2 seems to be [recommended](https://askubuntu.com/a/147266/43759) by many since it has [various advantages](https://wiki.archlinux.org/index.php/LVM#Advantages) including resizing of partitions even when they're mounted.  I've set aside the bulk (except 512 MiB for `/boot`) of SSD for LVM2.  The spinning disk is used for

* `/var`
* swap space

Once you've all the physical volumes that make up the LVM group, do

```
pvcreate /dev/nvme0n1p6
vgcreate lvmg1 /dev/nvme0n1p6
lvcreate -L 40G -n root lvmg1
lvcreate -l 100%FREE -n home lvmg1
```

Later on, `pacstrap` would automatically add the `root=` kernal parameter to map the `root` partition to the right logical volume.  This can be verified at `/boot/grub/grub.cfg`

```
linux /vmlinuz-linux root=/dev/mapper/lvmg1-root rw quiet
```

## Format

Once partitioned, make sure to format each of them. For swap

```
mkswap /dev/sda6
swapon /dev/sda6
```

Likewise for ext4 do

```
mkfs.ext4 /dev/lvmg1/root
```

## Mount

```
mount /dev/lvmg1/root /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p5 /mnt/boot
mkdir /mnt/home
mount /dev/lvmg1/home /mnt/home
mkdir /mnt/var
mount /dev/sda5 /mnt/var

mkdir /mnt/boot/efi
mount /dev/nvme0n1p1 /mnt/boot/efi
```

`genfstab` is a Arch script that setups up `/etc/fstab` as per the current mounts; it would later detect mounted file systems and swap space.  Since we're on a UEFI + GPT boot scheme, mount the EFI partition too for `genfstab` and `os-prober` to pick up Windows for GRUB to list Windows too.  This is the same partition as used by Windows for booting, mounted as `/mnt/boot/efi`.  Same goes for other Windows partitions that needs to be listed in `fstab` for auto-mounting in the new OS.  Otherwise it has to be [done manually](https://wiki.archlinux.org/index.php/Fstab) with Emacs and `lsblk -f` or `blkid`.

# Base Install

Before proceeding, it might be a good idea to fix the order of mirrors in `/etc/pacman.d/mirrorlist` since it's copied to the installed system as-is.

```
pacstrap /mnt base
```

# System Config

## fstab

```
genfstab -U /mnt >> /mnt/etc/fstab
```

## Change root

```
arch-chroot /mnt
```

Make sure to install packages that would be needed for network access in the installed system now; specifically `wpa_supplicant` since the new system wouldn't have network access and will not contain this package either.  This is a dependency for _netctl_.

## Time

Once done, know your time zone using `tzselect` and do

```
ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime
hwclock --systohc
```

## Locale

```
nano /etc/locale.gen
locale-gen
localectl set-locale LANG=en_IN.UTF-8
```

The last line sets `LANG`.

## Network

Set `/etc/hostname` and `/etc/hosts` for easy network identification.  Ping and check if the network configuration works fine.

# Update initramfs

Since we've used `lvm2` for `/` itself, it has to be added to the _HOOKS_ section of `mkinitcpio.conf`:

```
# /etc/mkinitcpio.conf
HOOKS="(base udev ... block lvm2 filesystems...)"
```

Regenerate initramfs image

```
mkinitcpio -p linux
```

# Root Password

Use `passwd` to set `root`'s password.

# Boot

Since Windows 8, Microsoft forces the UEFI + GPT combo, make sure this is followed by Arch for a successful coexistence with Windows.  Pre-installed Windows machines have a EFI partition already made from which Windows boots; verify this using `fdisk`.  [This means](https://wiki.archlinux.org/index.php/EFI_System_Partition#Mount_the_partition) we don't have to create an EFI partition but simply mount it for GRUB to use.  We're using GRUB as the bootloader.  The mounting part should've already been done as part of _§3.5 Mount_:

```
pacman -S --needed grub efibootmgr
mkdir /boot/efi
mount /dev/nvme0n1p1 /boot/efi
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
nano /etc/default/grub
grub-mkconfig -o /boot/grub/grub.cfg
```

The suggested _os-prober_ , which makes GRUB look for other OSs, didn't find Windows. Apparently, the EFI partition should be mounted to `/mnt/boot` _before_ pacstrapping as this is where the kernel and bootloader are installed to.  If you get `lvmetad` errors, you may safely ignore them.  Once inside the new Arch, os-prober should work fine.

# Reboot

```
umount -R /mnt
exit
umount -R /mnt
reboot
```

Secure boot doesn't work since it [needs a signed boot loader](https://wiki.archlinux.org/index.php/Secure_Boot#Using_a_signed_boot_loader); see [the forums](https://bbs.archlinux.org/viewtopic.php?id=202714) for details.  A few other errors

> mmc0: Unknown controller version (3). You may experience problems.
> mmc0: SDHCI controller on PCI [0000:03:00.0] using ADMA

However, this is [a false alarm](https://bugzilla.redhat.com/show_bug.cgi?id=1205070), and SD cards should read fine.

> kvm disabled by bios

Intel's virtualization hardware (VT-x) is disabled in BIOS and [hence this error](https://askubuntu.com/q/303164/).

> Failed unmounting /var

This also seems to be [a non-issue](https://bbs.archlinux.org/viewtopic.php?pid=1204644#p1204644).

## Network

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

