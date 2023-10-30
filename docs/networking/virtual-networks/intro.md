AppJail's Virtual Networks is a useful feature for creating private and isolated networks. A jail can be on multiple Virtual Networks at the same time. For example, it is possible to have a jail on one network for development and another for production.

!!! tip

    Organize your Virtual Networks using one of the following private spaces (RFC 1918):
    
    * 10.0.0.0/8
    * 172.16.0.0/12
    * 192.168.0.0/16

## Default Virtual Network (recommended)

!!! tip

    We recommend using the default network without creating a Virtual Network manually
    unless you have a specific need.

```sh
appjail quick hello \
    virtualnet=":hello" \
    overwrite \
    start
```

In the above case, `appjail quick` will run `appjail network auto-create` internally which will create a Virtual Network with some options defined in your AppJail configuration file, such as `AUTO_NETWORK_NAME` (default: `ajnet`) as its name, `AUTO_NETWORK_ADDR` (default: `10.0.0.0/10`) as its address and `AUTO_NETWORK_DESC` (default: `AppJail Network`) as its description. 

The `virtualnet` option is very interesting. As you can see it is defined as `:hello`. The double colon is necessary, because the network name has to be defined first, so when we use an "empty network name" it is just a hint for AppJail to create the default Virtual Network. After the double colon is the interface name, which in this case is `hello`. We can use some other special values such as:

* `<random>`: A random interface name is chosen. We recommend this option over others.
* `<name>`: The jail name is used. This option is recommended when planning to have only a few jails with unique names. This option can be problematic if we do not take into account that the jail name may have characters not allowed by an interface name. The length of the interface name must also be taken into account: 10 characters or less is preferable. 

An example using `<random>`:

```sh
appjail quick hello \
    virtualnet=":<random>" \
    overwrite \
    start
```

## Creating a Virtual Network Manually

To create a virtual network, we can use the `appjail network` command, but we need three things: network address, CIDR and the network name. The following example creates a network named `development` using `10.42.0.0` as the network address, and `24` as the CIDR.

```sh
appjail network add development 10.42.0.0/24
```

AppJail will load the following kernel modules before using this option:

* `if_bridge`
* `bridgestp`
* `if_epair`

Now, we can create a jail that is on the `development` network by simply using the `virtualnet` option.

```sh
appjail quick myjail \
    virtualnet="development:myjail" \
    overwrite \
    start
```

`myjail` is the interface name. This interface is created using cloning against `if_epair(4)`, so two pairs will be created for both the host and the jail.

We will create another jail that will be on the same network as `myjail` to show its communication using `ping(8)`.

```sh
appjail quick otherjail \
    virtualnet="development:otherjail" \
    overwrite \
    start
```

To obtain the IP address of each jail we can use `appjail network hosts`.

**myjail**:

```console
# appjail network hosts -REj myjail
10.42.0.2       development
```

**otherjail**:

```console
# appjail network hosts -REj otherjail
10.42.0.3       development
```

```console
# appjail cmd jexec otherjail ping -c4 10.42.0.2
PING 10.42.0.2 (10.42.0.2): 56 data bytes
64 bytes from 10.42.0.2: icmp_seq=0 ttl=64 time=0.175 ms
64 bytes from 10.42.0.2: icmp_seq=1 ttl=64 time=0.164 ms
64 bytes from 10.42.0.2: icmp_seq=2 ttl=64 time=0.165 ms
64 bytes from 10.42.0.2: icmp_seq=3 ttl=64 time=0.163 ms

--- 10.42.0.2 ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.163/0.167/0.175/0.005 ms
```

Instead of using `appjail network add` to create a new virtual network, you can specify one using the `network` option in `appjail quick`.

```sh
appjail quick jdns \
    network="dns 12.0.0.0/8" \
    virtualnet="dns:jdns default" \
    start \
    overwrite
```

This is useful for when you need to run a Makejail in multiple environments without resorting to `appjail network add`, but for simple use cases consider the [auto-created network](#creating-a-virtual-network-manually) using `appjail network auto-create`.
