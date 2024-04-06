The main motivation for combining the `alias` and `virtualnet` options is to provide a way for jails that do not use VNET, such as LinuxJails.

To use both options we need to create another loopback interface:

```sh
sysrc cloned_interfaces+="lo1"
sysrc ifconfig_lo1_name="appjail0"
service netif cloneup
```

The interface name can be whatever you want, but I recommend keeping it simple.

```sh
appjail quick debian \
    alias="appjail0" \
    virtualnet="development" \
    osversion=bullseye \
    type=linux+debootstrap \
    start \
    linuxfs \
    devfs_ruleset=0 \
    template=/tmp/linux.conf \
    overwrite
```

Because LinuxJails uses a loopback interface, we cannot communicate with jails that do not use the same communication method. If we really want to communicate between a jail that uses aliases and another jail that doesn't use aliases, we need to create the jail with a specific, static IPv4 address and write a small rule in our `pf.conf(5)` file.

```sh
appjail makejail \
    -j alpine \
    -f gh+AppJail-makejails/alpine-linux \
    -o template=/usr/local/share/examples/appjail/templates/linux.conf \
    -o alias \
    -o virtualnet=":appjail0 address:10.0.0.50 default" \
    -o nat
```

At this point we can only communicate with the outside, but with the following rule in our `pf.conf(5)`, we can communicate with other jails:

```
# Put this rule after the anchors you have configured.
nat on ajnet inet from 10.0.0.50 to 10.0.0.0/10 -> 10.0.0.1
```

Reload `pf(4)`'s rules:

```sh
service pf reload
```

If we send ICMP packets to other jails:

```console
# appjail cmd jexec alpine ping -c4 10.0.0.4
PING 10.0.0.4 (10.0.0.4): 56 data bytes
64 bytes from 10.0.0.4: seq=0 ttl=64 time=0.086 ms
64 bytes from 10.0.0.4: seq=1 ttl=64 time=0.057 ms
64 bytes from 10.0.0.4: seq=2 ttl=64 time=0.052 ms
64 bytes from 10.0.0.4: seq=3 ttl=64 time=0.057 ms

--- 10.0.0.4 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
# appjail cmd jexec alpine ping -c4 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=113 time=104.831 ms
64 bytes from 8.8.8.8: seq=1 ttl=113 time=129.898 ms
64 bytes from 8.8.8.8: seq=2 ttl=113 time=111.267 ms
64 bytes from 8.8.8.8: seq=3 ttl=113 time=105.241 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 104.831/112.809/129.898 ms#
```

Assuming that there is a jail with an IPv4 address `10.0.0.4`, as you can see, the communication is successful.

---

**See also**:

* [Virtual Networks](intro.md)
* [LinuxJails](../../linux.md)
* [Makejails](../../makejails/intro.md)
