+++
date = "2016-05-03T10:51:49+01:00"
title = "Disk backup using netcat and pv"
+++

Short summary on how to fully backup a disk over network using netcat and pv.

This solution is much faster than nfs or sftp but unencrypted. I experienced 112MB/s on my gigabit lan.


Server
======

```bash
netcat -l -w 2 1111 | pv -b > backup.img
```

Client
======

Use the `-s` flag to provide an estimated size for pv to calculate ETA.

```bash
pv -s250G -cN "sda" /dev/sda | nc 10.10.10.1 1111
```
