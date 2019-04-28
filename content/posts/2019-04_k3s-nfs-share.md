+++
date = "2019-04-28T12:00:00+01:00"
title = "K3s - NFS share"
+++

These are my personal notes while performing the installation.
Instructions might be incomplete, dangerous or wrong.

# Problem: Local storage for K3s installation in VM
Cleanest solution: Mount NFS on host and use hostpath on pod

Also: Would be good to export all k3s state, maybe backups are better than full nfs export

## Dom0 and K3s must communicate through link local network
```
dom0$ cat libvirt-net-private.xml
<network>
  <name>private</name>
  <bridge name='virbr1' stp='on' delay='0'/>
  <ip address='169.254.169.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='169.254.169.10' end='169.254.169.254'/>
    </dhcp>
  </ip>
</network>
```

```
virsh net-define ./libvirt-net-private.xml
virsh net-autostart private
virsh net-start private
```

Add to vbox
```
virsh edit k3s
    <interface type='bridge'>
      <source bridge='virbr1'/>
      <model type='virtio'/>
    </interface>
```

```
virsh shutdown k3s
virsh start k3s

# shows eth1
ip l

# configure network
auto eth1
iface eth1 inet static
    address 169.254.169.3
    netmask 255.255.255.0
    gateway 169.254.169.1
```

```
/etc/init.d/networking restart
```

Ping 169.254.169.1 does not work from k3s
Check arp on dom0 (works):
```
$ arp -i virbr1
169.254.169.3            ether   52:54:00:f0:60:24   C                     virbr1
```

My guess is that firewall is preventing traffic.
```
dom0$ iptables -P INPUT ACCEPT
```
Ping works -> add iptable rules

Add to firewall definition:
```
IVMS="virbr1"
iptables -A INPUT -i $IVMS -j ACCEPT
```

# Add NFS share

/etc/fstab:
```
UUID=7a8730cf-4185-4d56-b1e9-d053a7b17a50 /mnt/red1_vms 	btrfs 	rw,noatime,nodiratime,compress=lzo,space_cache,subvol=@vms 0 0
/mnt/red1_vms/storage/k3s				  /export/k3s_store	  none 	  bind 		   0 0
```

/etc/exports
```
/export/k3s_store       169.254.169.0/255.255.255.0(rw,sync,no_subtree_check)
```

```
# Reload nfs
exportfs -ra
```

```
k3s$ mount 169.254.169.1:/export/k3s_store /mnt/nfs
# Does not work connection refused
k3s$ apk add nfs-utils
k3s$ mount 169.254.169.1:/export/k3s_store /mnt/nfs
# works now
```

Add correct fstab automount: /etc/fstab
```
169.254.169.1:/export/k3s_store /mnt/k3s_store nfs auto,rw,_netdev 0 0
```

Startup automount
```
$ rc-update add nfsmount
$ rc-service nfsmount start
$ reboot
```

# NFS permission denied on k3s

Checklist
- Mounted with rw
- Exported with rw
- dom0 can write on it

Problem with mount directory is root owned and client-root is trying to write at it.
With NFSs default setting root_squash (man exports), the root user is mapped to another uid.

We could disable the root uid remapping:
```
/export/k3s_store       169.254.169.0/255.255.255.0(rw,sync,no_root_squash,no_subtree_check)
```

This handles the k3s root user the same as the dom0 root user.
Another approach is to chmod a+rwx the desired directory and face the situation, that uid/gid 0 from the guest is mapped to nobody:nobody on the filesystem.
Since many containers are running as root and security is not a real concern here, we disable root squashing with no_root_squash for now.

# Side-note: Host directory sharing

Dom0:
```
<filesystem type='mount' accessmode='mapped'>
  <source dir='/mnt/red1_vms/storage/k3s'/>
  <target dir='k3s_store'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
</filesystem>
```

Guest:
```
k3s_store /mnt/k3s_store            9p             trans=virtio    0       0
```

Does not work as expected, because file permissions are mapped. Only virtualized filesystem. Problems with k3s setup (I/O errors).

