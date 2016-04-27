---
layout: post
title: "Multiple Squid Instances and Multiple Subnets"
date: 2016-03-07 11:00:00
tags: ['squid', 'proxy', 'linux']
description: "One of my company's client was running a Squid Proxy/Cache for two different branches with two Multiple Squid Instances bound to two different ports. Everythin was running smoothly, until he opened the following request: I need to track down how much bandwith each branch office is using, if possible, by the proxy internal IP address that must be inside the branch's VLAN. To solve this problem, I now had to run not just in different ports, but also in different IPs going out different gateways. That should be fun, and ineed was."
comments: True
---

One of my company's client was running a Squid Proxy/Cache for two different branches with two Multiple Squid Instances bound to two different ports. Everythin was running smoothly, until he opened the following request:
> I need to track down how much bandwith each branch office is using, if possible, by the proxy internal IP address that must be inside the branch's VLAN.

To solve this problem, I now had to run not just in different ports, but also in different IPs going out different gateways. That should be fun, and ineed was.

## The scenario
Here is a draft of the scenario:     

```
eth0: VLAN 10, IP 10.10.10.100/24, Gateway 10.10.10.1
eth1: VLAN 20, IP 10.10.20.100/24, Gateway 10.10.20.1
```

## One Linux box, two gateways
The biggets difficult was to route the traffic from each subnet into their default gateways. The basic network configuration allowed just one of them, sending all traffic from both VLANs into the same destination, which only worked for one subnet.     
To work this out, I needed to create custom routes with `iproute2`.     

First, create a new route policy, appending the content below in `/etc/iproute2/rt_table`:

```bash
echo "1 admin" >> /etc/iproute2/rt_tables
```

Here's the configuration in `/etc/network/interfaces`:

``` bash
auto eth0
iface eth0 inet static
        address 10.10.10.100
        netmask 255.255.255.0
        gateway 10.10.10.1

auto eth1
iface eth1 inet static
        address 10.10.20.100
        netmask 255.255.255.0
```

And the configuration used in `iproute2`, you can put in `/etc/rc.local`:

```bash
ip route add 10.10.20.0/24 dev eth1 src 10.10.20.100 table admin
ip route add default via 10.10.20.1 dev eth1 table admin
ip rule add from 10.10.20.100/32 table admin
ip rule add to 10.10.20.100/32 table admin
ip route flush cache
```

To test the network, I pinged to an exeternal address from each interface, like:

```bash
ping -c1 -I eth0 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 10.10.10.100 eth0: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=54 time=20.4 ms

--- 8.8.8.8 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 20.419/20.419/20.419/0.000 ms
```

```bash
ping -c1 -I eth1 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 10.10.20.100 eth1: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=54 time=20.4 ms

--- 8.8.8.8 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 20.419/20.419/20.419/0.000 ms
```

## Squid configuration
To get multiple Squids, I installed one from package management and one compiling the source, this way I don't get conflict in configuration parameters or directories. If you want to compile two separated Squid instances, take a look at [Squid Multiple Instances documentation](http://wiki.squid-cache.org/MultipleInstances).    
In each Squid instance, I bound the IP address of the interface to the daemon with different ports. Also, I forced all trafic to go out by that IP, like:

```bash
http_port 10.10.10.100:3128
tcp_outgoing_address 10.10.10.100
```

```bash
http_port 10.10.20.100:3129
tcp_outgoing_address 10.10.20.100
```

We now have what we needed! All traffic from each proxy is leaving from a specifc IP. To monitor its traffic, we configured a NAT rule to a static public address on our firewall, allowing us to measure the bandwidth.     

The `iproute2` configuration used was posted by **Darien Kindlund** in this [blog](https://kindlund.wordpress.com/2007/11/19/configuring-multiple-default-routes-in-linux/).
