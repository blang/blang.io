+++
date = "2015-11-18T20:51:49+01:00"
title = "Migrate etcd 0.4.x to etcd 2.2"
+++

I updated my 'never update' coreos server and what could i tell you, it was a bad idea (update often or never).
Disclaimer: If you have a running 0.4.x instance and want to update, this might not be the best guide for you.
I made the mistake to update before backing up etcd, this guide is about recovering all your data from a ".ss" file, found in the old `etcd` data directory (found at `/var/lib/etcd/`).

<!--more-->

Migrate your `.ss` file to a snapshot
========================

First problem: You could not restore a cluster from a data directory in `0.4.x`. If you try to start a `0.4.x` server, this node can not join the cluster an refuses to work. [Issue](https://github.com/coreos/etcd/issues/863)

Well then step up the version ladder, but slowly:

Get the `etcd-v0.5.0-alpha.5-linux-amd64.tar.gz` from the [github releases](https://github.com/coreos/etcd/releases/tag/v0.5.0-alpha.5).

Migrate your `0.4.x` data directory to `0.5.x`:

```bash
./etcd-migrate -data-dir="./data/etcd" -name="1426dada90d8448d816aac1358181a1a"
```

If you don't know your effin etcd name anymore and the migrate tool does not help you, search the `etcd/log` file for something. If found mine because of a internal coreos entry.

Get access again
================

Now you have a proper `.snap` file inside `etcd/snap/` something like `000...60.snap`.
Sadly i could not import this into etcd, but i thought at least i can run a `0.5.x` server with it.

Adjust your ips:
```bash
./etcd --listen-client-urls=http://192.168.0.131:4001 -data-dir ./data/etcd -name e1 --listen-peer-urls=http://192.168.0.131:7001 --advertise-client-urls=http://192.168.0.131:4001
```
You might want to add `-force-new-cluster`.

At least you can access your data again:
```bash
./etcdctl --debug --peers http://192.168.0.131:4001 ls /
```

Migrate to 2.0.0
================

Remove your `etcd/wal` directory. Download etcd `2.0.0` from the [github release page](https://github.com/coreos/etcd/releases/tag/v2.0.0).

```
./etcd-migrate -data-dir="./data/etcd" -name="1426dada90d8448d816aac1358181a1a"
```
Yes i had to do this again, maybe you could use the `2.0.0` migration without the `0.5.0`, i don't know.

Start your `2.0.0` server to test everything is fine:

```bash
./etcd --listen-client-urls=http://192.168.0.131:4001 -data-dir ./data/etcd -name e1 --listen-peer-urls=http://192.168.0.131:7001 --advertise-client-urls=http://192.168.0.131:4001
```

Transfer to 2.2
===============

Transfer your data directory to your `2.2` server (to `etcd2` directory and let the first start migrate itself. On Coreos you might simply want to start the service with the proper configuration:
```
systemctl start etcd2
```

Or manually but remember the set all cluster flags to avoid resetting the config again.
```bash
etcd2 --data-dir /var/lib/etcd2
```

Backups
=======

Now, better create backups:
```
./etcdctl backup --data-dir ./data/etcd --backup-dir=./bkp
```

