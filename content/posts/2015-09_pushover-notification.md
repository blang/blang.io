---
title: Pushover notification for long running tasks
date: 2015-09-14
comments: true
---

Last day i had to wipe the disk of my vserver and since this can take a long time, i needed a way to get notified when the process is done.

I need an easy notification mechanism from any shell connected to the internet. 

The solution i found was the notification api of [pushover.net](https://pushover.net). It provides a very simple api to create Push-Notifications for Android or IOS.

It's simple:

- Create an account
- Download Android/iPhone App
- Register an app to get an api token

Now how to get notified if the disk shredding is complete?

You could get a small shell script to do the job, but i wanted something that works without hassle on every platform:

I wrote [pushovercli](https://github.com/blang/pushovercli) in golang. 
It's quite an overkill, but does the job and i do not need to worry if `curl` or the correct shell is installed.

Get notified
------------

First, download the binary that suits your system from the [releases list](https://github.com/blang/pushovercli/releases).

Now you're ready to go:
```bash
$ export PUSHOVER_USER=[my user token]
$ export PUSHOVER_TOKEN=[my app token]
$ shred /dev/sdX && ./pushovercli "Shred complete" || ./pushovercli "Shred failed"
```
