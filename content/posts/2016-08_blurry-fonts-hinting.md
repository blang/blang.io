+++
date = "2016-08-29T10:51:49+01:00"
title = "Remove blurry console fonts - Hinting"
+++

Small problems, especially regarding fonts, can drive you crazy over time.

Hinting is such a thing, i was troubleshooting many hours since i could not state what's wrong. The font simply felt wrong, really hurt my eye.

After comparing the font with exact same settings with my old gentoo installation and taking many screenshots i found the problem: Hinting.

By default, there's a config `/etc/fonts/conf.d/10-hinting-slight.conf` on arch and many other distros.

It removes the crisp and joy from my fonts in urxvt, here's a comparision to see it for yourself.
Pay attention to the letter `m` for example.

Simply remove the symlink from `conf.d` and restart X, the difference is mind blowing.

![alt text](/posts/2016-08_blurry-fonts-hinting.png "Font hinting")
