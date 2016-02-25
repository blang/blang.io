+++
date = "2016-02-25T14:51:49+01:00"
title = "Create Windows 7 bootable installation usb"
+++

This is how you create a bootable usb to install Windows 7 (NON EFI):

Attention, make sure you understand every step before executing it. If not used correctly can result in fatal dataloss.

Assumptions:

- USB stick with ~4GB: /dev/sdb (change device according to your setup, don't overwrite your second harddisk!)

Get the iso
===========

Visit http://mirror.corenoc.de/digitalrivercontent.net/
and download one via torrent

Wipe the device
===============

If you're like me and tried to `dd` the iso to your usb stick or created multiple partition tables in this process, better wipe your stick.

```
wipefs -a /dev/sdb
dd if=/dev/zero of=/dev/sdb bs=512 count=1 conv=notrunc
```

Partition and format
====================

Delete everything, create one partition with ~4GB, MBR table, bootable flag.

```
fdisk /dev/sdb
```

Format using ntfs:

```
mkfs.nfts /dev/sdb1

# Create a label for use in the bootloader
nftslabel /dev/sdb1 winboot
```

Copy the data
=============

```
mount -o loop ~/windows.iso /mnt/iso
mount /dev/sdb1 /mnt/stick
cp -rv /mnt/iso/* /mnt/stick/
```

Install grub as bootloader
==========================

Note: Keep `/dev/sdb1` still mounted 
```
grub-install --target=i386-pc --boot-directory="/mnt/stick/boot" /dev/sdb
```

Configure grub
==============

Edit `/mnt/stick/boot/grub/grub.cfg`:

```
default=1
timeout=15

menuentry "Installation" {
    insmod ntfs
    insmod search_label
    search --no-floppy --set=root --label winboot --hint hd0,msdos1
    ntldr /bootmgr
}

menuentry "First hard drive" {
    insmod ntfs
    insmod chain
    insmod part_msdos
    insmod part_gpt
    set root=(hd1)
    chainloader +1
    boot
}
```

Unmount and Installation
========================

```
umount /dev/sdb1
```

And now, try to boot the stick. Have fun!
