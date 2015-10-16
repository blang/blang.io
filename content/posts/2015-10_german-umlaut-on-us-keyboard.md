+++
date = "2015-10-16T09:05:52+02:00"
title = "German umlaute on us keyboard"
+++

The use of an english keyboard layout is a good choice for every programmer since all brackets and important symbols are far better reachable than on the german keyboard layout, but there's one problem, the umlaute.


<!--more-->

Umlaute and the english layout
------------------------------

It's quite tedious to use something like `setxkbmap de` to write a german email and change back if there's any code in sight.

But there's a really simple and beautiful solution: `Xmodmap`

Using Xmodmap you can customize your keyboard layout, but simply adding those umlaute would not do the trick. The best option is to use `Capslock` as a modifier key.

Then, you can use `Capslock + a` to print an `ä` for example.

- `Caps Lock + a` -> `ä`
- `Caps Lock + u` -> `ü`
- `Caps Lock + o` -> `ö`
- `Caps Lock + s` -> `ß`
- `Caps Lock + t` -> `€` 

Add `Shift` to write upper case.

~/.Xmodmap
-------

Here's the config:

```
keycode 66 = Mode_switch Multi_key
keycode 39 = s S ssharp
keycode 38 = a A adiaeresis Adiaeresis
keycode 30 = u U udiaeresis Udiaeresis
keycode 32 = o O odiaeresis Odiaeresis
keycode 28 = t T EuroSign EuroSign
```

Activate it using `xmodmap ~/.Xmodmap`
