---
title: Resize luks partition under lvm
date: 2015-03-19
comments: true
---

A simple collection of commands used to resize a partition encrypted by luks and resize lvm volumes on top of that.

Make sure you understand everything before using any command below, this might destroy all your data!

Umount all volumes, deactivate lvm `lvchange -a n vg0` and close luks.

Delete partition you want to expand, recreate partition with same start and different ending.

# Extend luks
Now, here are some steps to follow

```
cryptsetup luksOpen /dev/sdxx lukslvm
lvchange -a n vg0 # disable lvm
cryptsetup status lukslvm # Check current size
cryptsetup resize lukslvm
cryptsetup status lukslvm # Check new size
cryptsetup luksClose lukslvm
cryptsetup luksOpen /dev/sdxx lukslvm
pvs # Check physical volume size
pvresize /dev/mapper/lukslvm
pvs # Check new physical volume size
lvs # check logical volume sizes
lvextend --size +1000M /dev/vg0/home
lvs # check new size
```

# Extend filesystem
Extend size of your filesystem on `/dev/vg0/home`, e.g.:

* ext4 `resize2fs -p /dev/vg0/home`
* xfs `xfs_growfs /mntpoint`


# Shrinking of filesystem
For shrinking on xfs you need to copy everything

```
rsync -aAXvH /mnt/root/* /mnt/backup/root # Backup
lvreduce --size -20G /dev/vg0/root # Resize
mkfs.X /dev/vg0/root 
rsync -aAXvH /mnt/backup/root/* /mnt/root # Restore
```