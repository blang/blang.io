---
title: CoreOS Qemu Image - Extend Docker graph size
date: 2015-03-16
comments: true
---

CoreOS ships by default with a runnable qemu image which works quite nicely. One problem i encountered fairly early is the lack of disk space.

If the reason for this is a big docker graph the solution might be to move your docker graph entirely. A nice side effect is the ability to persist your graph and optimize I/O rate by using direct disk access.

-------

# Create a seperate disk for dockers graph

I'm using lvm volumes for this:

```
lvcreate -L 100G -n core0docker vg0
```

# Add the disk to your startup

Add the disk to your machine startup. In my case it's my `virsh` config:

```
<disk type='block' device='disk'>
  <driver name='qemu' type='raw' io='native'/>
  <source dev='/dev/mapper/vg0-core0docker'/>
  <target dev='vdc' bus='virtio'/>
</disk>
```

Reboot your coreos machine to get the new device.

# Systemd mount

Create a systemd mount file `media-docker.mount`:

```
[Unit]
Wants=user-configvirtfs.service
Before=user-configvirtfs.service
# Only mount config drive block devices automatically in virtual machines
# or any host that has it explicitly enabled and not explicitly disabled.
ConditionVirtualization=|vm
ConditionKernelCommandLine=|coreos.configdrive=1
ConditionKernelCommandLine=!coreos.configdrive=0

# Support old style setup for now
Wants=addon-run@media-configvirtfs.service addon-config@media-configvirtfs.service
Before=addon-run@media-configvirtfs.service addon-config@media-configvirtfs.service

[Mount]
What=/dev/vdc
Where=/media/docker
Options=rw
Type=ext4

[Install]
WantedBy=multi-user.target
```

Activate the new mount service:

    cp media-docker.mount /etc/systemd/system/
    systemctl enable /etc/systemd/system/media-docker.mount
    systemctl start media-docker

# Move your docker graph

    rsync -aXS /var/lib/docker/. /media/docker/dockergraph/

We're using rsync to handle docker sparse files correctly.

# Setup docker to use new graph directory


    cp /usr/lib/systemd/system/docker.service /etc/systemd/system/


Add the `-g` option to `ExecStart` to change docker graph directory:


    ExecStart=/usr/lib/coreos/dockerd --daemon --host=fd:// $DOCKER_OPTS $DOCKER_OPT_BIP $DOCKER_OPT_MTU $DOCKER_OPT_IPMASQ -g /media/docker/dockergraph


Restart your docker service:

    systemctl daemon-reload
    systemctl restart docker


Now you're done and you can remove your old graph.


    rm -rf /var/lib/docker/