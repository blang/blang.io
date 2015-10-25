+++
date = "2015-10-25T11:33:57+01:00"
title = "Yubikey FIDO U2F on Gentoo"
+++

I just got the github promo [Yubikey with U2F](https://www.yubico.com/products/yubikey-hardware/fido-u2f-security-key/).
It wasn't as plug and play as i thought so i will share my experience.


<!--more-->
The Error
=========

Visiting the [yubico demo site](https://demo.yubico.com/u2f?tab=register) i got the useless error:
`Exception: FIDO Client error: 1 (OTHER ERROR)`

Kernel: HIDraw
==============

First of all, your kernel needs `HIDRAW=y`:
```
Device Drivers -> HID support -> /dev/hidraw raw HID device support
```

UDEV Rules
==========

First get the [u2f udev rules from github](https://github.com/Yubico/libu2f-host/blob/master/70-u2f.rules) and download to `/etc/udev/rules.d/70-u2f.rules`.

After that you need additional udev rules to permit access for a regular user:
```
KERNEL=="hidraw*", SUBSYSTEM=="hidraw", MODE="0664", GROUP="plugdev", ATTRS{idVendor}=="1050", ATTRS{idProduct}=="0113|0114|0115|0116|0120"
```

Reload udev rules and re-plug your yubikey to be sure.
```
udevadm control --reload-rules
```

Check `/dev/hidraw*` for group permissions: `root:plugdev`.

Add usergroup
=============

This will add the group `plugdev` in charge.

Add your user to the `plugdev` group:
```
gpasswd -a username plugdev
```

You might want to reboot/relogin and verify you're in the plugdev group.

Test yubikey
============

Go to [yubicos demo site](https://demo.yubico.com/u2f?tab=register) and try your yubikey.
If you still get the error recheck permissions, use `lsusb` and `dmesg` to check your key is available.

