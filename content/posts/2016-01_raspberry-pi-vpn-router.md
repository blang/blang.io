+++
date = "2016-01-13T14:51:49+01:00"
title = "Raspberry PI VPN Router"
+++

In this little tutorial i explain how to build a raspberry pi vpn router.
This raspi will be equiped with an additional ethernet adapter and work as a router between your pc and your home lan or between portions of your local network.
The raspi will use NAT to connect devices connected to eth1 to your ISP or a choice of multiple vpns.
What was important for me is to ensure that your device only passes the selected output channel. If the vpn breaks down, no internet even if the second vpn or your isps connection still works. No auto switch to your isp.

Requirements:

- USB Ethernet adapter
- Linux skills
- Rasperry pi with fresh raspbian

<!--more-->

How does it work?
=================

Your eth0 is connected to your local lan and gets his routing via dhcp. Your eth1 is connected to your computer.
On eth1 dnsmasq serves the 192.168.111.X network of vpn routable devices.

Basically we are enable NAT routing on the raspi and by default null-route everything incoming.
After this, we create a separate routing table `defgw` with your eth0 default gateway aquired by dhcp.

We enable NAT on every tunX for our vpns.
Every VPN connection will add a separate routing table `vpnX` with your raspi as gateway.

Now you can decide to assign an ip out of the private pool `192.168.111.X` to a routing table.

If you choose `defgw` it gets routed to your local network and out to your isp.

If you choose `vpnX` it gets routed through the specified vpn.

If a route breaks or gets removed, the null-route rule take care of it instead falling back to your default gatework (ISP).

Install software
========================

Packages:

- openvpn
- dnsmasq

Routing
=======

Add to /etc/iproute2/rt_tables:
```
200 vpn1
201 vpn2
152 null
153 defgw
```

Network setup
=============

/etc/network/interfaces
```
source-directory /etc/network/interfaces.d

auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
    post-up /home/pi/ifup.sh

auto eth1
iface eth1 inet static
    address 192.168.111.1
    netmask 255.255.255.0
    network 192.168.111.0
    broadcast 192.168.111.255
```

The up-script sets up the `defgw` route.

 /home/pi/ifup.sh
```
#!/bin/bash
if [ "$IFACE" != "eth0" ]; then
    echo "No suitable interface"
    exit 0
fi

gw=$(ip route show dev eth0 | grep default | head -n 1 | awk '{print $3}')
if [ -z "$gw" ]; then
    echo "No default gateway found"
    exit 0
fi

ip route add default via $gw dev eth0 table defgw

exit 0
```

Startup
=======

/etc/rc.local
```
echo "Enable null routing"
ip route add default via 127.0.0.1 dev lo table null
ip rule add from 192.168.111.0/24 table null
ip rule add from 192.168.111.1

echo "Enable NAT on eth0"
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE


# NAT on tunX
echo "Enable NAT on tunX"
iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE
iptables -t nat -A POSTROUTING -o tun1 -j MASQUERADE
iptables -t nat -A POSTROUTING -o tun2 -j MASQUERADE
iptables -t nat -A POSTROUTING -o tun3 -j MASQUERADE

echo "Starting vpn service 1"
start-stop-daemon --start --background --make-pidfile --oknodo --user root --name vpn1 --pidfile /var/run/vpn1.pid --startas /home/pi/vpn1.sh --chuid root -- --daemon
echo "Starting vpn service 2"
start-stop-daemon --start --background --make-pidfile --oknodo --user root --name vpn2 --pidfile /var/run/vpn2.pid --startas /home/pi/vpn2.sh --chuid root -- --daemon

exit 0
```

VPN
=====

/home/pi/vpnX.sh
```
#!/bin/bash
openvpn --cd /home/pi/.openvpn --config yourconfig.ovpn --up up.sh --script-security 2 --route-nopull
```

You need a seperate up script for every openvpn connection, change the routing table vpn1 to your need.

So your vpns are clearly distinguished by routing table and not device.

/home/pi/.openvpn/up.sh:
```
#!/bin/sh
ip route add default via $ifconfig_local dev $dev table vpn1
```

Switch the routing table
========================

For demonstration purpose i use a connected device with ip `192.168.111.33`:

Switch to vpn1:
```
ip rule add from 192.168.111.33 table vpn1
Remove afterwards:
ip rule del from 192.168.111.33 table vpn1
```

Switch to vpn2:
```
ip rule add from 192.168.111.33 table vpn2
Remove afterwards:
ip rule del from 192.168.111.33 table vpn2
```

Switch to your isp route:
```
ip rule add from 192.168.111.33 table defgw
Remove afterwards:
ip rule del from 192.168.111.33 table defgw
```
