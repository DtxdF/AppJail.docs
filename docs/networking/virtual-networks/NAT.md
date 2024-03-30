Thanks to virtual networks, AppJail can use NAT in two ways: manually, using `appjail nat` or automatically using `appjail quick`.

!!! warning

    You must read [Packet Filter](../packet-filter.md) before using this option.

```sh
appjail quick jnat \
    virtualnet="development:jnat default" \
    nat \
    start
# explicitly
appjail quick jnat \
    virtualnet="development:jnat default" \
    nat="network:development" \
    overwrite \
    start
```

The `nat` option requires the `network` parameter to be defined, but as you can see in the example above, the virtual network `development` is the default network, which implies that the gateway is used as the default router, also, as `appjail quick` knows, it uses that network implicitly in some options like `nat` and `expose`.

You can apply NAT on multiple networks. This is useful only if you plan to use the source address for different purposes.

```console
# appjail quick jnat \
    virtualnet="development:jn1 default" \
    virtualnet="web:jn2" \
    nat \
    nat="network:web" \
    start \
    overwrite
...
# appjail network hosts -REj jnat
192.128.0.2     development
10.0.0.2        web
# appjail cmd jexec jnat ping -c4 -S 192.128.0.2 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 192.128.0.2: 56 data bytes
64 bytes from 8.8.8.8: icmp_seq=0 ttl=114 time=45.209 ms
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=44.204 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=44.436 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=44.864 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 44.204/44.678/45.209/0.387 ms
# appjail cmd jexec jnat ping -c4 -S 10.0.0.2 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 10.0.0.2: 56 data bytes
64 bytes from 8.8.8.8: icmp_seq=0 ttl=114 time=44.984 ms
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=45.167 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=44.426 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=44.581 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 44.426/44.789/45.167/0.298 ms
```

To create NAT rules manually for an existing jail, we can use `appjail nat`.

```sh
appjail quick jnat \
    virtualnet="development:jn1 default" \
    overwrite
appjail nat add jail -n development jnat
appjail-config set -Ij jnat exec.prestart='appjail nat on jail ${name}'
appjail-config set -Ij jnat exec.poststop='appjail nat off jail ${name}'
appjail start jnat
```

AppJail can apply NAT to an entire network space instead of a single jail. To do this, we need to use `appjail nat network` in the same way AppJail does for jails manually.

```console
# appjail nat add network db
# appjail nat on network db
# appjail nat boot on network db
# service appjail-natnet status
NAT Information:

BOOT  NAME  RULE
1     db    nat on "jext" from 10.42.0.0/24 to any -> ("jext:0")

Status:

nat on jext inet from 10.42.0.0/24 to any -> (jext:0) round-robin
```

The `appjail nat boot on network` command sets the boot flag to apply NAT rules when starting FreeBSD using the `appjail-natnet` service.

!!! tip

    To apply NAT to an entire network at startup, enable the RC script:

    ```sh
    sysrc appjail_natnet_enable=YES
    ```

When applying NAT rules for networks, it can be useful to apply NONAT for a single jail to avoid masking its IP address. AppJail can do this in the same way as NAT rules.

```sh
# Using appjail quick:
appjail quick jnonat \
    virtualnet="db:jnonat default" \
    nonat \
    overwrite \
    start
# Manually:
appjail quick jnonat \
    virtualnet="db:jnonat default" \
    overwrite
appjail nat add jail -n db -N jnonat
appjail-config set -Ij jnonat exec.prestart='appjail nat on jail ${name}'
appjail-config set -Ij jnonat exec.poststop='appjail nat off jail ${name}'
appjail start jnonat
```

!!! info

    Since IP forwarding is enabled, jails that are on different networks don't get
    isolation. This is not a problem when isolation is not necessary in such a way,
    you can use the firewall to block some packets, but you can also use virtual
    networks with IP forwarding enabled simply to get better organization.
    Consider using [Bridges](../bridges/intro.md) when possible.

!!! warning

    If `EXT_IF` and `ON_IF` are configured to a vtnet device (e.g., vtnet0), you may
    experience extremely slow download speed inside the jail. That might relate to a
    checksum issue described [here](https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=277718).
    You can set `hw.vtnet.csum_disable="1"` in /boot/loader.conf to make CPU to
    perform the checksum instead of your NIC to solve the problem temporarily.

---

**See also**:

* [Virtual Networks](intro.md)
