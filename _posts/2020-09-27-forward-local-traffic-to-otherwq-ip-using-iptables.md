---
layout: post
title: Forward local traffic to other ip using iptables 
description: Forward local traffic to other ip using iptables
---
First of all you have to enable `route_localnet` for the kernel to not consider source loopback address as 
"martian" while routing, see: [ip-sysctl](https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt).
```
$ sudo sysctl -w net.ipv4.conf.all.route_localnet=1
```

Let's transparently send all udp dns traffic to `1.1.1.1` instead.
```
$ iptables -t nat -A OUTPUT -p udp --dport 53 -j DNAT --to-destination 1.1.1.1:53
$ iptables -t nat -A POSTROUTING -j MASQUERADE
```

Or send all traffic targeting port `65000` in localhost to `192.168.0.1:80`
```
$ sudo iptables -t nat -A OUTPUT -p tcp --dest 127.0.0.1 --dport 65000 -j DNAT --to-destination 192.168.0.1:80
$ sudo iptables -t nat -A POSTROUTING -j MASQUERADE
```

Worth to mention that OUTPUT chain will have effect only for traffic originating locally.

