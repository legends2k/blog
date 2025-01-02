+++
title = "Arch Linux Installation"
description = "coexisting with Windows on a dual GPU laptop"
date = "2018-05-27T17:00:46-07:00"
tags = ["tech", "tools", "linux"]
toc = true
+++

I've a dual GPU laptop: _Lenovo Legion Y520-15IKBN_; the _Nvidia GeForce 1050 Ti Mobile_ for graphics-heavy (3D) tasks and _Intel HD Graphics 630_ for rendering the rest.  It has an SSD and a traditional spinning disk.  UEFI + GPT as opposed to the classic BIOS + MBR combo is preferable.  I'm a long-time Xubuntu user but I wanted to try Arch Linux.  Here I note my experience in getting this laptop up and running.

# Prelims

A few things upfront --- detailed later --- to decide if this guide is for you

1. **Secure boot** off: at least for (USB) installation.  Longer term, there're some hoops to jump through but you can enable it back on
2. **SSD vs non-SSD**: Use spinning disks for volatile data
3. **SATA vs RAID**: To even detect your M.2 PCIe SSD the storage controller is to be set to ACHI and not Intel RST
4. **LVM**: if you want the flexibility of changing partition sizes without data loss, post installation, even while mounted

Being a Emacs junkie one of the first things I do when I start using a new environment is remap my <kbd>CAPSLOCK</kbd> to <kbd>CONTROL</kbd>.

## Ctrl your CapsLk

[Map `CAPSLOCK` to `CTRL`](https://www.emacswiki.org/emacs/MovingTheCtrlKey#toc8) with `loadkeys` [just for the current session](https://wiki.archlinux.org/index.php/Keyboard_configuration_in_console)

{{< highlight basic >}}
dumpkeys | head -1 > keys.map
echo 'keycode 58 = Control' >> keys.map
loadkeys keys.map
rm keys.map
{{< /highlight >}}

Once we’ve a desktop environment setup, this can be made permanent with [`.Xmodmap`](https://bitbucket.org/rmsundaram/tryouts/src/dev/Misc/config/.Xmodmap).

# Wireless Network

Without internet access modern linux installations are an exercise in pain.  I didn't have an ethernet cable so I had to get wi-fi up:

1. If `ping` fails, disable _dhcpcd_ service with `systemctl stop dhcpcd@` (press <kbd>TAB</kbd>)
2. Check wireless interface creation with `iw dev`; it should list the interface name e.g. `wlp3s0`
    * If absent, `lspci -v` to ensure the wireless network controller drivers loaded right
    * `ip link show` shows the networking interfaces present
3. Check if wireless is not blocked with `rfkill list`
    * If hard-blocked, press hardware button to enable wireless hardware
    * If soft-blocked: `rfkill unblock wifi`
4. Configure wireless using `iwctl`
    * `device list` (check station name which is usually `wlan0`)
    * `station wlan0 connect SSID`

# Partitions

## SSD

Check if the drive got detected `ls /dev/nv*`.  If not, the SSD is probably configured to use the RST (RAID) storage controller which Linux doesn't seem to detect.  It has to be AHCI (SATA) controller.

If the default Windows installation was done with RST selection, Windows 10 will not boot if this switch is flipped.  Recovering Windows without a reinstallation [is possible](http://triplescomputers.com/blog/uncategorized/solution-switch-windows-10-from-raidide-to-ahci-operation/).  Minimal _Safe Mode_ can be enabled/disabled without using `bcdedit` [through GUI](http://www.thewindowsclub.com/reboot-in-safe-mode-windows) too: `msconfig`.  [Tests show](https://www.reddit.com/r/Dell/comments/4gke4k/a_closer_look_at_ahci_vs_raid/) that using the older AHCI doesn't incur any significant performance hit over RAID.

## Scheme

SSDs are [not recommended](https://unix.stackexchange.com/a/89230) for volatile data, so `/var` and swap space should be allocated on spinning disk.  Rest like `/boot`, `/`, `/usr`, etc. can be left on the SSD: seen under `/dev/nvme0nXpY` not `/dev/sda`; `X` is the disk number, `Y` is the partition.

**Note**: `/boot` needn’t be in a separate partition while `/boot/efi` should be; the [EFI System Partition][esp] partition is mounted in `/boot/efi`.  `/boot` on its own partition, outside LVM, additionally has the advantage of [GRUB environment block][grub-save]; useful if you use `GRUB_SAVEDEFAULT` for GRUB to remember last OS selection.

Partition with `cfdisk` and verify them with `lsblk` or `fdisk -l`.  Choose _Linux filesystem_ for _Partition Type_ for everything except swap and LVM.  Mind that these partitions are the ones created outside the LVM; if you're going to manage all your Linux partitions with LVM, just create one partition (in addition to [ESP][]) and set its type to _Linux LVM_ in `fdisk`.

My final scheme (sublist intelligible after reading the next section)

1. `/boot` → `/dev/nvme0n1p5` (512 MiB)
2. LVM → `/dev/nvme0n1p6` (54.5 GiB)
    1. `/` → `/dev/lvmg1/root` (40 GiB)
    2. `/home` → `/dev/lvmg1/home` (remaining)
3. `/var` → `dev/sda5` (4 GiB)
4. swap → `dev/sda6` (4 GiB)

[esp]: https://en.wikipedia.org/wiki/EFI_system_partition
[grub-save]: https://www.gnu.org/software/grub/manual/grub/html_node/Environment-block.html#Environment-block

## LVM

Using LVM2 seems to be [recommended](https://askubuntu.com/a/147266/43759) by many since it has [various advantages](https://wiki.archlinux.org/index.php/LVM#Advantages) including resizing of partitions even when they're mounted.  I've set aside the bulk (except 512 MiB for `/boot`) of SSD for LVM2.  The spinning disk is used for

* `/var`
* swap space

Once you've all the physical volumes that make up the LVM group, do

{{< highlight basic >}}
pvcreate /dev/nvme0n1p6
vgcreate lvmg1 /dev/nvme0n1p6
lvcreate -L 40G -n root lvmg1
lvcreate -l 100%FREE -n home lvmg1
{{< /highlight >}}

Later on, `pacstrap` would automatically add the `root=` kernal parameter to map the `root` partition to the right logical volume.  This can be verified at `/boot/grub/grub.cfg`

{{< highlight basic >}}
linux /vmlinuz-linux root=/dev/mapper/lvmg1-root rw quiet
{{< /highlight >}}

## Format

Once partitioned, make sure to format each of them. For swap

{{< highlight basic >}}
mkswap /dev/sda6
swapon /dev/sda6
{{< /highlight >}}

Likewise for ext4 do

{{< highlight basic >}}
mkfs.ext4 /dev/lvmg1/root
{{< /highlight >}}

## Mount

{{< highlight basic >}}
mount    /dev/lvmg1/root /mnt
mount -m /dev/nvme0n1p5  /mnt/boot
mount -m /dev/nvme0n1p1  /mnt/boot/efi
mount -m /dev/lvmg1/home /mnt/home
mount -m /dev/sda5       /mnt/var
{{< /highlight >}}

`genfstab` is a Arch script that setups up `/etc/fstab` as per the current mounted file systems and swap space.  Since we're on a UEFI + GPT boot scheme, mount the EFI partition too for `genfstab` and GRUB2 (`os-prober`) to pick up Windows.  This is the same partition as used by Windows for booting, mounted as `/mnt/boot/efi`.  Same goes for other Windows partitions that needs to be listed in `fstab` for auto-mounting in the new OS.  Otherwise it has to be [done manually](https://wiki.archlinux.org/index.php/Fstab) with Emacs and `lsblk -f` or `blkid`.

#### Reserved Space

I came to know that on ext4 [Linux silently reserves][ext4-reserve] 5% of system partitions for the root user; to be able to log in, if a regular user fills up the drive.  [See the reservation][tunefs] thus

{{< highlight basic >}}
> tune2fs -l /dev/sda5 | egrep "Reserved block count|Block size" | paste -sd\ | awk '{print ($4 * $7 / ( 1024 * 1024 ) ), "MiB"}'
204.797 MiB
{{< /highlight >}}

You can change this with `tune2fs -m <percentage> <device>`; mine’s too petty to change.

[ext4-reserve]: https://unix.stackexchange.com/q/7950/30580
[tunefs]: https://superuser.com/q/444269/50345

# Base Install

Before proceeding, it might be a good idea to check `/etc/pacman.d/mirrorlist`; for me, it was already setup with the optimal mirror by a background systemd unit once network was up :)  Check it’s [mtime][] and contents.  If needed, update using [`reflector`][]; it retrieves latest mirror list based on country:

{{< highlight basic >}}
pacman -S --needed reflector
reflector --verbose --country IN -l10 --sort rate --save /etc/pacman.d/mirrorlist
{{< /highlight >}}

Proceed with the base installation:

{{< highlight basic >}}
pacstrap -K /mnt base base-devel linux linux-firmware linux-headers acpi lvm2 grub efibootmgr os-prober iwd sudo exfatprogs intel-ucode smartmontools lshw inxi strace rsync wget mc nano emacs rg fd fzf bfs 7zip pv git links tree
{{< /highlight >}}

It’s important to include packages needed for the booted system to be self-sufficient.  For instance, missing out wireless networking package will leave you with no internet connectivity!
I usually go for a well-rounded commandline system.

[`reflector`]: https://www.ostechnix.com/retrieve-latest-mirror-list-using-reflector-arch-linux/
[mtime]: https://en.wikipedia.org/wiki/Stat_(system_call)

# System Config

## fstab

{{< highlight basic >}}
genfstab -U /mnt >> /mnt/etc/fstab
{{< /highlight >}}

Usually `/var` is given additional options `nodev,nosuid,noexec`; while `/home` is marked with `nodev` for [tighter security][why-nodev-nosuid].

Check `/mnt/etc/fstab`, the table of filesystems and their mount points, the system refers for auto-mounting at every boot.

[why-nodev-nosuid]: https://serverfault.com/q/547237/139619

## Change root

{{< highlight basic >}}
arch-chroot /mnt
{{< /highlight >}}

## Time

Once done, know your time zone using `tzselect` and do

{{< highlight basic >}}
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
hwclock --systohc
{{< /highlight >}}

## Locale

{{< highlight basic >}}
nano /etc/locale.gen
locale-gen

{{< /highlight >}}

The last line sets `LANG`.

## Network

Set `/etc/hostname` for easy network identification.  Ping and check if the network configuration works fine.

# Update initramfs

Since we've used `lvm2` for `/` itself, it has to be added to the _HOOKS_ section of `mkinitcpio.conf`:

{{< highlight cfg >}}
# /etc/mkinitcpio.conf
HOOKS="(base udev ... block lvm2 filesystems...)"
{{< /highlight >}}

Regenerate `initramfs` image

{{< highlight basic >}}
mkinitcpio -P
{{< /highlight >}}

# Root Password

Use `passwd` to set `root`'s password.

# Boot

Since Windows 8, Microsoft forces the UEFI + GPT combo, make sure this is followed by Arch for an amicable coexistence with Windows.  Pre-installed Windows machines have a EFI partition already made from which Windows boots; verify this using `fdisk`.  [This means](https://wiki.archlinux.org/index.php/EFI_System_Partition#Mount_the_partition) we don't have to create an EFI partition but simply mount it for GRUB to use.  We're using GRUB as the bootloader.  The mounting part should've already been done as part of _§3.5 Mount_:

{{< highlight basic >}}
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
nano /etc/default/grub
grub-mkconfig -o /boot/grub/grub.cfg
{{< /highlight >}}

# Reboot

{{< highlight basic >}}
exit
umount -R /mnt
reboot
{{< /highlight >}}

Secure boot doesn't work since it [needs a signed boot loader](https://wiki.archlinux.org/index.php/Secure_Boot#Using_a_signed_boot_loader); see [the forums](https://bbs.archlinux.org/viewtopic.php?id=202714) for details.  A few other errors

{{< highlight basic >}}
mmc0: Unknown controller version (3). You may experience problems.
mmc0: SDHCI controller on PCI [0000:03:00.0] using ADMA
{{< /highlight >}}

However, this is [a false alarm](https://bugzilla.redhat.com/show_bug.cgi?id=1205070), and SD cards should read fine.

{{< highlight basic >}}
kvm disabled by bios
{{< /highlight >}}

Intel's virtualization hardware (VT-x) is disabled in BIOS and [hence this error](https://askubuntu.com/q/303164/).

{{< highlight basic >}}
Failed unmounting /var
{{< /highlight >}}

This also seems to be [a non-issue](https://bbs.archlinux.org/viewtopic.php?pid=1204644#p1204644).

# Processor Microcodes

Processor security patches are released by CPU manufacturers, to be released as motherboard firmware updates; these come in slowly (or don’t 😉).  Linux solves this by loading the microcode update as early as possible during boot.

Check boot log (`journalctl -b0`) for CPU bugs messages

{{< highlight txt >}}
MDS CPU bug present and SMT on, data leak possible. See https://www.kernel.org/doc/html/latest/admin-guide/hw-vuln/mds.html for more d>
MMIO Stale Data CPU bug present and SMT on, data leak possible. See https://www.kernel.org/doc/html/latest/admin-guide/hw-vuln/process>
{{< /highlight >}}

Refer [Microcode - ArchWiki][aw-microcode] to check if your processor has patches and perform necessary steps to install and load at boot.

{{< highlight basic >}}
pacman -S --needed intel-ucode
grub-mkconfig -o /boot/grub/grub.cfg
{{< /highlight >}}

[aw-microcode]: https://wiki.archlinux.org/title/Microcode

# Hibernation

Hibernation needs a resume parameter in kernel with the disk to retrieve data from.  Check swap partition UUID:

{{< highlight basic >}}
swaplabel /dev/sda6
{{< /highlight >}}

Edit `/etc/default/grub` and append the parameter

{{< highlight cfg >}}
GRUB_CMDLINE_LINUX_DEFAULT="... resume=UUID=YOUR-SWAP-PARTITION-UUID"
{{< /highlight >}}

You also need the `resume` hook (a script run on the [initial ramdisk][]) in `/etc/mkinitcpio.conf`; ensure it’s added _after_ `udev` and `lvm2`; I added it almost the last:

{{< highlight cfg >}}
HOOKS=(base udev .. lvm2 .. resume fsck)
{{< /highlight >}}

Generate the ramdisk image and GRUB’s configuration file:

{{< highlight basic >}}
mkinitcpio -P

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
{{< /highlight >}}

Hibernate (from Xfce4 panel’s Action Button) and resume should work now.  [Xfce 4.20+ _Power Manager_ supports _Hybrid Sleep_][xfce-hybrid-sleep] and rightly [inhibits systemd’s ACPI settings][acpi-inhibit] 😁.

[acpi-inhibit]: https://wiki.archlinux.org/title/Power_management#Power_managers
[initial ramdisk]: https://en.wikipedia.org/wiki/Initial_ramdisk
[xfce-hybrid-sleep]: https://xfce.org/download/changelogs/4.20

# Up next...

This completes my Arch Linux installation process.  The newly installed Arch now has be configured and maintained 😊.  The [Arch Linux Configuration][] article covers

* Xfce4 desktop environment
* Dual GPU set up
* Enlisting Windows in boot menu
* Auto mounting USB sticks
* Pulseaudio
* Bluetooth
* Laptop Mode Tools

and more.

[Arch Linux Configuration]: {{< relref "arch_config.md" >}}
