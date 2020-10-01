+++
title = "Resize root under LVM"
description = "Shrink root, expand home!"
date = 2020-06-22T07:36:47+05:30
tags = ["tech", "tools", "linux"]
+++

My root partition had unused space while `/home` was starving.  I needed to _resize both without data loss_.  I’d foreseen this and created my Linux partitions using [LVM][] while [installing Arch](/note/arch_install#lvm) 😇 .

Since operations here involve resizing `/`, **boot from Live USB** and then perform them.  Also make sure the USB is **UEFI bootable**; otherwise it’d mean changing BIOS settings to enable _Legacy Boot_.  *This may lead to GRUB’s disappearance from boot menu.*

[LVM]: https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)

# Shrink `/`

{{< highlight basic >}}
# Pick-up LVM volumes
sudo vgchange -a y

# verify volume groups
sudo vgs

# check file system
sudo e2fsck -fy /dev/lvmg1/root

# verify logical volumes
sudo lvdisplay

# Resize file system to size smaller than volume size
sudo resize2fs /dev/lvmg1/root 29G

# Resize logical volume to desired size
sudo lvreduce -L 30G /dev/lvmg1/root

# Refill file system to volume size
sudo resize2fs /dev/lvmg1/root

# Verify shrunk size in logical volumes
sudo lvdisplay

# Verify if free space shows up in volume group
sudo vgdisplay
{{< /highlight >}}

# Expand `/home`

{{< highlight basic >}}
# Expand logical volume to all of free space
sudo lvextend -l +100%FREE /dev/lvmg1/home

# Verify grown size in logical volumes
sudo lvdisplay

# Check file system
sudo e2fsck -fy /dev/lvmg1/home

# Grow file system to fill up volume
sudo resize2fs /dev/lvmg1/home
{{< /highlight >}}

Overall LVM resizing experience: _smooth_!

# References

1. [Decrease an LVM Partition](https://www.rootusers.com/lvm-resize-how-to-decrease-an-lvm-partition/)
2. [Increase an LVM Partition](https://www.rootusers.com/lvm-resize-how-to-increase-an-lvm-partition/)

[LVM]: https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)
