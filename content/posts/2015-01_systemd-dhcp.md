---
title: Systemd Unit Network DHCP not ready
date: 2015-01-18
comments: true
---

After installing coreos on a hetzner server i realized that fleet unit files pulling from docker fail on boot, because there was no dhcp lease yet.

```
dial tcp: lookup index.docker.io: connection refused
```

`journalctl` shows that dhcp gets an ip after starting the container unit file, therefore `docker pull` fails.

-----

## Solution

First you have to add a network dependency to your unit file:

```
Wants=docker.service etcd.service network-online.target 
After=docker.service etcd.service network-online.target 
```

But the `network-online.target` does not work unless you enable another service first:

```
systemctl enable systemd-networkd-wait-online.service
```

This sadly does not work on current CoreOS stable `522.5.0` but on beta `557.0.0`