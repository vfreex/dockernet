Dockernet
=========

Dockernet enables you to build a flexible [Docker][] network.

With Dockernet, you can add custom interfaces, configure static IP addresses, and adding static routes to a Linux container in a simple way.

Developer: Rayson Zhu <vfreex@gmail.com>

This project is inspired from [pipework][].

Content Table
-------------

- [Things to note](#things-to-note)
- [Add an extra interface](#add-an-extra-interface)
- [Set a static IP address](#set-a-static-ip-address)
- [Add a static route](#add-a-static-route)

Things to note
--------------
1. Use the `--net=none` option when start a container to tell Docker not to configure its network automatically, leaving you free to build any of the custom configurations explored in the last few sections of this document. For example, start a [CentOS][] container without configuring the network:
```bash
$ sudo docker run -ti --rm --name=test_container --net=none centos:latest /bin/bash
```
2. If you want to start a process after extra network interface is up in a container, you can add `dockernet` script to your Docker image and call `dockernet --wait <interface>`, like `dockernet --wait eth0`. It will wait until that interface is present and in UP operational state.

Add an extra interface
----------------------
Use `dockernet <container> addif <interface> [<bridge>=docker0]` to add a extra interface to a container.

Following command will add interface `eth1` to container `test_container` and bridge this interface with default interface `docker0`:

```bash
sudo dockernet test_container addif eth1
```

If you want to bridge your newly created interface to another interface `br0` other than `docker0`, issue:

```bash
sudo dockernet test_container addif eth1 br0
```

Then, set operational state to UP:

```bash
sudo dockernet test_container up eth1
```

Set a static IP address
-----------------------
Usually, you should clear all IP addresse associated with that interface in advance:

```bash
dockernet <container> clearip <interface>
```

Then, add an IP address. You should specify an IP address in [CIDR notation][]. `<default gateway>` is optional.

```bash
dockernet <container> addip <interface> <CIDR> [<default gateway>]
```

For instance, we add IPv4 address `192.0.2.100/24` and IPv6 address `2001:db8:1234:5678::100/64`
with default gateways `192.0.2.1` and `2001:db8:1234:5678::1` to interface `eth1` for container `test_container`:

```bash
sudo dockernet test_container clearip eth1
sudo dockernet test_container addip eth1 192.0.2.100/24 192.0.2.1
sudo dockernet test_container addip eth1 2001:db8:1234:5678::100/64 2001:db8:1234:5678::1
```

Add a static route
------------------
You can use following notation to add or delete a route. The `<ROUTE>` part given in these commands are directly passed to the `ip route` command.

```bash
dockernet <CONTAINER> addroute <CONTAINER_IFNAME> <ROUTE>
dockernet <CONTAINER> delroute <CONTAINER_IFNAME> <ROUTE>
```

For instence, let's add 2 static routes to container `test_container`:

```bash
sudo dockernet test_container addroute eth1 203.0.113.0/24 via 192.0.2.254 metric 1000
sudo dockernet test_container addroute eth1 2001:db8:abcd::/48 via 2001:db8:1234:5678::ffff metric 1000
```
These two rule will tell your container that route packets targeting network `203.0.113.0/24` to router `192.0.2.254` and `2001:db8:abcd::/48` to router `2001:db8:1234:5678::ffff` through interface `eth1`.


[Docker]: https://www.docker.com/
[pipework]: https://github.com/jpetazzo/pipework
[CentOS]: https://www.centos.org/
[CIDR notation]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#CIDR_notation
