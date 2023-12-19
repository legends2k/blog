+++
title = "Move Linux b/w Drives"
description = "From HDD to SSD"
date = 2023-12-18T13:00:00+05:30
tags = ["tech", "tools", "linux"]
toc = true
+++

Flash memories comes in different forms and use different interfaces to connect.  Pen drives use [flash memory][] while
SSDs use [solid state storage][sss].  SSDs can be interfaced with your [mobo][] either through AHCI (SATA) or PCIe; M.2
is actually a form factor (standard stating about the size and form of such drives which are small and rectangular;
looks like RAM modules compared to the 2.5 inch drives) and is orthogonal to the interface.  More importantly, there are
M.2 drives connecting over AHCI (typically slower); go for PCIe interfaced M.2 drives if you want speed.  These drives
work using the [NVMe protocol][nvme]; for details [refer][ssd-diffs].

I recently bought a 500 GB NVMe drive for my AMD Ryzen 5600 desktop.  Thanks to the spinning 3.5 inch HDD which was
audibly grinding most of the time, the machine didn’t realize its full potential.  Now I love the software setup on this
machine, with Arch at the helm and everything setup the way I want.

This is a write-up on moving the OS setup from the HDD to SSD; it uses Linux Logical Volume Manager (LVM2) to
flexibility (lossless partition sizes updates).

[sss]: https://en.wikipedia.org/wiki/Solid-state_storage
[flash memory]: https://en.wikipedia.org/wiki/Flash_memory
[mobo]: https://en.wikipedia.org/wiki/Motherboard
[nvme]: https://en.wikipedia.org/wiki/NVM_Express
[ssd-diffs]: https://www.pcworld.com/article/558324/nvme-vs-m-2-vs-sata-ssd-whats-the-difference.html

# Setup NVMe LBA

Install NVMe CLI (`nvme`) for setting the right [LBA][] size; it can do a bunch of other things:

{{< highlight basic >}}
yay -S --needed nvme-cli

nvme id-ns -H /dev/nvme0n1 | grep "Relative Performance"

LBA Format  0 : Metadata Size: 0   bytes - Data Size: 512 bytes - Relative Performance: 0x1 Better (in use)
LBA Format  1 : Metadata Size: 0   bytes - Data Size: 4096 bytes - Relative Performance: 0 Best
{{< /highlight >}}

Looks like the LBA format isn’t set for best performance:

{{< highlight basic >}}
nvme format --lbaf=1 /dev/nvme0n1

You are about to format nvme0n1, namespace 0xffffffff(ALL namespaces).
WARNING: Format may irrevocably delete this device's data.
You have 10 seconds to press Ctrl-C to cancel this operation.

Use the force [--force] option to suppress this warning.
Sending format operation ...
Success formatting namespace:ffffffff
{{< /highlight >}}

This fixed the LBA for best performance.  `nvme` has a bunch of other useful options; try

{{< highlight basic >}}
nvme list
nvme id-ctrl -H /dev/nvme0
nvme smart-log /dev/nvme0
nvme fw-log /dev/nvme0
{{< /highlight >}}

[LBA]: https://en.wikipedia.org/wiki/Logical_block_addressing

# Fixed Partitioning

EFI needs its own partition (ESP) for at least 512 MiB [\[3\]](#references).  The rest is made into one partition; note the
types:

{{< highlight basic >}}
Disk /dev/nvme0n1: 465.76 GiB, 500107862016 bytes, 122096646 sectors
Disk model: CT500P3SSD8
Units: sectors of 1 * 4096 = 4096 bytes
Sector size (logical/physical): 4096 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: C95F77F1-0464-4676-8BE3-97325DEA12A1

Device          Start       End   Sectors   Size Type
/dev/nvme0n1p1    256    131327    131072   512M EFI System
/dev/nvme0n1p2 131328 122096639 121965312 465.3G Linux LVM
{{< /highlight >}}

## Format ESP

Ensure that ESP is FAT32 for wider recognition; logical sector size is 4096 as LBA is too; cluster size couldn’t be any
larger than 1.

{{< highlight basic >}}
sudo mkfs.vfat -F 32 -S 4096 -s 1 -v -n EFI /dev/nvme0n1p1

mkfs.fat 4.2 (2021-01-31)
/dev/nvme0n1p1 has 64 heads and 32 sectors per track,
hidden sectors 0x0800;
logical sector size is 4096,
using 0xf8 media descriptor, with 131072 sectors;
drive number 0x80;
filesystem has 2 32-bit FATs and 1 sector per cluster.
FAT size is 128 sectors, and provides 130784 clusters.
There are 32 reserved sectors.
Volume ID is 387acba5, volume label EFI.
{{< /highlight >}}

Formatting the _Linux LVM_ partition is redundant as we’ll make logical volumes on it and anyway format them later.

# Logical Partitioning

Usual drill: create physcial volume (PV), volume group (VG) and logical volumes (LV) under VG.  For this case PV is just
the NVMe device and there’s just going to be one VG; refer [\[4\]](#references) for various other configurations.

As per [\[5\]](#references)

{{< highlight basic >}}
pvcreate /dev/nvme0n1p2
vgcreate nvme /dev/nvme0n1p2
lvcreate -L 40G -n root nvme
lvcreate -L 16G -n var nvme
lvcreate -L 16G -n home nvme
lvcreate -L 200G -n data nvme
lvcreate -L 197896M -n media nvme
{{< /highlight >}}

Use `pvs`, `vgs` and `lvs` for short, and `pvdisplay`, `vgdisplay` and `lvdisplay` for detailed view of the layout.

`lsblk -f` shows the final partitioning scheme

{{< highlight basic >}}
nvme0n1
├─nvme0n1p1    vfat        FAT32    EFI
└─nvme0n1p2    LVM2_member LVM2 001
  ├─nvme-root
  ├─nvme-var
  ├─nvme-home
  ├─nvme-data
  └─nvme-media
{{< /highlight >}}

## File System

Format logical volumes individually with required filesystems.  Ensure that reserved space isn’t too much:

{{< highlight basic >}}
# Set directly while creating (to 1%)
mkfs.ext4 -m 1 /dev/mapper/nvme-media

# Do it after formatting
# Display reserved space
tune2fs -l /dev/mapper/nvme-data | grep -E "Reserved block count|Block size" | paste -sd\ | awk '{print ($4 * $7 / ( 1024 * 1024 ) ), "MiB"}'
2048 MiB

# Get block size
tune2fs -l /dev/mapper/nvme-data | grep 'Block size:'

# Tweak it with gigs * 1024^3 / block size (4096 usually)
tune2fs -r 524288 /dev/mapper/nvme-media 
{{< /highlight >}}

Enable `fast_commit` [\[6\]](#references): `tune2fs -O fast_commit /dev/mapper/nvme-root` and verify.

# Copy Data to SSD

Boot into SystemRescue Live OS from a USB; I simply copied all files in its ISO to a FAT32 USB; UEFI boot simply looks for a `EFI` directory.
Label it `RESUCE1002` as the loader looks for a filesystem by this label to get its data once booted; without this you'd still get a shell but
without much tools.

{{< highlight basic >}}
# Mount old and new partitions for /boot/efi, / and /home
mount -m /dev/sda1 /mnt/old/esp
mount -m /dev/nvme0n1p1 /mnt/new/esp

# Sync: mind the slash and the lack thereof; we want to copy the contents (not the directory)
rsync -qaHAXS /mnt/old/root/ /mnt/new/root
{{< /highlight >}}

The parameters come from ArchWiki; refer [\[7\]](#references).

# Fix `/etc/fstab`

Use `blkid` and fix UUIDs of new partitions. Using `genfstab` is an option but I fixed it manually.

# Reinstall GRUB

Unmount everything

{{< highlight basic >}}
umount /mnt/old/{esp,root,home}
umount /mnt/new/{esp,root,home}
rmdir /mnt/{old,new}
{{< /highlight >}}

Change root

{{< highlight basic >}}
mount -m /dev/mapper/nvme-root /mnt
mount -m /dev/mapper/nvme-home /mnt/home
mount -m /dev/nvme0n1p1 /mnt/boot/efi

arch-chroot /mnt
{{< /highlight >}}

Install GRUB; enable `lvm` and set `GRUB_DISABLE_OS_PROBER` as don't to probe and list old HDD's OSes.

{{< highlight basic >}}
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB

# Add `lvm` to GRUB_PRELOAD_MODULES
nano /etc/default/grub

grub-mkconfig -o /boot/grub/grub.cfg
{{< /highlight >}}

Regenerate initramfs image after adding `lvm2` to `HOOKS` in `/etc/mkinitcpio.conf` after `block` but before `filesystem`:

{{< highlight basic >}}
nano /etc/mkinitcpio.conf

mkinitcpio -p linux
{{< /highlight >}}

# Appendix A: Remove Stale `/dev/mapper/*`

Fixed partitioning (`fdisk`) first and logical partitioning (LVM) next; I forgot this basic rule and flipped the order.

It isn’t advised to go with no fixed partitions.  `/boot/efi` is  better off as a plain partition than a logical one
[\[2\]](#references) and also some software might consider the device unpartitioned; risky [\[1\]](#references)!  This is about
reverting it.

{{< highlight basic >}}
lsblk -pf

/dev/nvme0n1
├─/dev/mapper/nvme-root
├─/dev/mapper/nvme-var
├─/dev/mapper/nvme-home
├─/dev/mapper/nvme-media
└─/dev/mapper/nvme-data
{{< /highlight >}}

showed this.  I quickly partitioned the device (with `fdisk`) as [GPT][] with two partitions assuming the LVM-stuff would
get overwritten.  It did but I ended up with this zombie.  Yikes!

{{< highlight basic >}}
/dev/nvme0n1
├─/dev/mapper/nvme-root
├─/dev/mapper/nvme-var
├─/dev/mapper/nvme-home
├─/dev/mapper/nvme-media
├─/dev/mapper/nvme-data
├─/dev/nvme0n1p1
└─/dev/nvme0n1p2
{{< /highlight >}}

Removing the partition table didn’t fix anything other than dropping the last two entries.  `pvs`, `vgs`, nor `lvs`
maintained that there’s nothing.  The following low-level LVM command fixed the situation.

{{< highlight basic >}}
sudo dd if=/dev/zero of=/dev/nvme0n1 bs=8K count=1 oflag=direct
sudo dmsetup remove /dev/dm-{0,1,2,3,4}
{{< /highlight >}}

[gpt]: https://en.wikipedia.org/wiki/GUID_Partition_Table

# References

1. [Create partition table (`fdisk`) before LVM2](https://unix.stackexchange.com/questions/195643/is-fdisk-needed-with-lvm)
2. [LVM Why Is It Not Recommended to Put the Boot Partition on LVM](https://www.baeldung.com/linux/lvm-boot-partition-recommendations)
3. [Absolute Minimum ESP Size](https://superuser.com/questions/1310927/what-is-the-absolute-minimum-size-a-uefi-system-partition-can-be)
4. [LVM - Wikipedia](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux))
5. [Advanced Format - ArchWiki](https://wiki.archlinux.org/title/Advanced_Format)
6. [Ext4 - ArchWiki](https://wiki.archlinux.org/title/Ext4)
7. [Migrating Arch from SSD to NVMe](https://code.saghul.net/2022/12/migrating-an-archlinux-install-from-an-ssd-to-an-nvme-drive/)

# See Also

1. [Move `/var` and `/home` to a separate NVME partition](https://unix.stackexchange.com/questions/701200/move-var-and-home-directory-on-a-separate-nvme-partition)
