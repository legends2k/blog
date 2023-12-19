+++
title = "GRUB Recovery"
description = "after a Windows boot loader overwrite"
date = 2019-01-20T18:35:20+05:30
tags = ["tech", "linux"]
+++

I tried upgrading the firmware of my old iOmega 1 TB HDD.  Internally it’s a _Barracuda LP_ family, ST31000520AS model Segate HDD.  Seagate had released a [firmware upgrade][] for this drive in 2010.  I’m around 9 years late to the party!

My laptop doesn’t have a CD-ROM drive, also I prefer burning the ISO to a USB stick --- why waste a CD, eh?  [Rufus][] kindly refused as the supplied Seagate update ISO’s boot format was unrecognizable.  In the process, I’d tried using another tool -- _RMPrepUsb_ -- in vain.  It’d done something, owing to some option I chose of course, to overwrite GRUB2 with Windows boot loader --- some `BCDedit` command I guess.  My meddling with BIOS didn’t help either as GRUB wasn’t even listed in _EFI order_, so I reverted to orignial settings[^1]:

| Setting     | Value                                            |
| ----------- | ------------------------------------------------ |
| Boot Mode   | UEFI                                             |
| USB Boot    | Enabled                                          |
| EFI order   | GRUB, Windows Boot Manager, EFI PXE Network      |
| Secure Boot | Disabled                                         |
| Fast Boot   | Enabled                                          |

Thanks to [Debian’s GrubEFIReinstall guide][]; it made me recall that the _Boot Mode_ should be UEFI; it also gave the way to verify it.  I’d to use a UEFI-bootable[^2] USB stick with Arch Linux ISO.  Things were smooth thereon!

{{< highlight basic >}}
root@archiso / # [ -d /sys/firmware/efi ] && echo "EFI boot on HDD" || echo "Legacy boot on HDD"
EFI boot on HDD
root@archiso / # mount /dev/lvmg1/root /mnt
root@archiso / # mount /dev/nvme0n1p5 /mnt/boot
root@archiso / # mount /dev/lvmg1/home /mnt/home
root@archiso / # mount /dev/sda5 /mnt/var
root@archiso / # swapon /dev/sda6
root@archiso / # arch-chroot /mnt
{{< /highlight >}}

Once [chroot][]ed[^3], I’d to reinstall GRUB to fix the issue!

{{< highlight basic >}}
> mount /dev/nvme0n1p1 /boot/efi
> grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
{{< /highlight >}}

All is well now 😀

# GRUB missing in UEFI’s NVRAM

On an ArchLinux box (host) with UEFI + GPT setup, I’d to install Ubuntu on an external SSD, for a machine having legacy BIOS (target).  I booted host in _Legacy Boot_ mode and installed Ubuntu by marking the first primary partition on the SSD _bootable_; everything went well.  Rebooting showed GRUB on the SSD with both Ubuntu and ArchLinnux.  Since target machine won’t have Arch, I’d to turn off os-prober and regenerate GRUB configuration file [^4]:

{{< highlight basic >}}
# cat >> /etc/default/grub
GRUB_DISABLE_OS_PROBER=true

# update-grub
{{< /highlight >}}

However, after switching back to _UEFI Boot_, I only saw _Windows Boot Manager_ and _EFI PXE Network_ options; GRUB was missing!  Mounting the EFI System Partition (ESP) showed GRUB under `/EFI/GRUB/grubx64.efi`, so it’s just the NVRAM GRUB was removed from.  Reinstalling GRUB by booting from an Arch Linux ISO is an option, but I didn’t want to lose my GRUB customizations like wallpaper, etc.  [Thanks to a good Unix.SE answer][efibootmgr-grub] helped me put GRUB back in NVRAM

{{< highlight bash >}}
# -c creates an entry with -l taking path in ESP at /dev/nvme0n1p1
efibootmgr -c -d /dev/nvme0n1 -p 1 -l \\EFI\\GRUB\\grubx64.efi -L GRUB
{{< /highlight >}}

# See Also

* [Debian’s GrubEFIReinstall guide][] -- like most Debian documents, thorough!
* [GRUB2 & EFI Recovery Tutorial][] seems to be a detailed guide with multiple methods; I only glanced it.
* [Firmware update Seagate HDD using Linux][] uses `hdparm` to do it on Linux; something the OEM doesn’t support.  I didn’t have to resort to it.  I dunno if this would’ve worked if the HDD was connected via USB.

----------

# HDD Upgrade Story

Those still reading to know about the poor ’ol HDD’s fate, read on!  It ends well :)

I created a plain _Free DOS_ bootable USB with Rufus; extracted the IMA floppy image to root; this worked since the update ISO was also using Free DOS.  The tool, booted with Free DOS, still couldn’t do the deed!  It couldn’t detect the drive that was connected via USB; [I realised][ManualFirmwareUpgrade] that it expects BIOS to be set to ATA; neither AHCI nor RAID -- the only options mine has.  

Luckily my 2006 desktop (AMD Phenom X4 machine) had the SATA option (in addition to the other two)!  I didn’t even have to resort to a force firmware write, the tool did it straight-forward.  I didn’t connect it via USB though; I opened the iOmega case, connected the internal HDD directly to the motherboard’s SATA power and data connectors.

Before firmware upgrade `smartctl` warned about an older firmware.

{{< highlight basic "hl_lines=10" >}}
> smartctl -i /dev/sdb
smartctl 7.0 2018-12-30 r4883 [x86_64-linux-4.20.3-arch1-1-ARCH] (local build)
Copyright (C) 2002-18, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Model Family:     Seagate Barracuda LP
Device Model:     ST31000520AS
Serial Number:    XXXXXXXX
LU WWN Device Id: X XXXXXX XXXXXXXXX
Firmware Version: CC32
User Capacity:    1,000,204,886,016 bytes [1.00 TB]
Sector Size:      512 bytes logical/physical
Rotation Rate:    5900 rpm
Device is:        In smartctl database [for details use: -P show]
ATA Version is:   ATA8-ACS T13/1699-D revision 4
SATA Version is:  SATA 2.6, 3.0 Gb/s
Local Time is:    Mon Jan 21 09:08:22 2019 IST
SMART support is: Available - device has SMART capability.
SMART support is: Enabled

==> WARNING: A firmware update for this drive may be available,
see the following Seagate web pages:
http://knowledge.seagate.com/articles/en_US/FAQ/207931en
http://knowledge.seagate.com/articles/en_US/FAQ/213915en
{{< /highlight >}}

After the upgrade, the warning vanished and _Firmware Version_ reads `CC35` 😇

[Rufus]: https://rufus.akeo.ie/
[firmware upgrade]: http://knowledge.seagate.com/articles/en_US/FAQ/213915en
[efibootmgr-grub]: https://unix.stackexchange.com/a/475245/30580
[Debian’s GrubEFIReinstall guide]: https://wiki.debian.org/GrubEFIReinstall
[chroot]: https://en.wikipedia.org/wiki/Chroot
[ManualFirmwareUpgrade]: https://niallbest.com/seagate-2tb-st32000542as-cc35-firmware-upgrade/
[GRUB2 & EFI Recovery Tutorial]: https://www.dedoimedo.com/computers/grub2-efi-recovery.html
[Firmware update Seagate HDD using Linux]: https://github.com/jandelgado/general/wiki/Firmware-update-of-Seagate-harddisk-using-Linux

[^1]: GRUB is present in _EFI Order_ since this was made after the recovery.
[^2]: This is rather important; a legacy boot USB stick wouldn’t be bootable when _Boot Mode_ is UEFI.
[^3]: This is important too; without chrooting file paths would be off.
[^4]: Modern machines have `grub-mkconfig` instead of `update-grub`
