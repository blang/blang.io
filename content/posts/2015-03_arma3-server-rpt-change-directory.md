---
title: ArmA3 Server RPT - Change directory of logs
date: 2015-03-17
comments: true
---

By default, arma server rpts (internal log files) reside inside `%AppData%\Local\Arma 3`.

If you're running multiple server instances like me, that's quite a mess.

# Change log directory
First, create a directory `logs_server1` inside your Arma3 Server base-directory.

Now you can use the `-profiles` flag to change the directory:

    arma3server.exe -server -port=4102 -profiles=logs_server1