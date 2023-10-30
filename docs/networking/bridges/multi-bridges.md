Virtual networks are useful, but another useful feature of AppJail is bridges, as described in previous sections.

This is useful when you want to have better control of the address pool using a DHCP server such as `dns/dnsmasq` or for some more specific application.

```sh
# jpub
appjail quick jpub \
    bridge="jpb1 iface:jext bridge:public" \
    bridge="jpb2 bridge:private" \
    mount_devfs \
    devfs_ruleset=10 \
    dhcp="sb_jpb1" \
    dhcp="sb_jpb2" \
    overwrite
# jpriv
appjail quick jpriv \
    bridge="jpv bridge:private" \
    mount_devfs \
    devfs_ruleset=10 \
    dhcp="sb_jpv" \
    overwrite 
```

`jpub` is on both `private` and `public` bridges, so it can connect to the outside and can communicate with other jails on the `private` bridge. `jpriv` cannot make connections to the outside, so we need a private DHCP server.

```sh
# AppJail can create a bridge if it does not exist, but dnsmasq requires the interface to have an IP address.
ifconfig bridge create name private
ifconfig private inet 129.0.0.1/24
dnsmasq --interface=private --dhcp-range=129.0.0.2,129.0.0.150,12h -d
```

Jails are started and should have their IP addresses.

```console
# appjail start jpub
...
# appjail cmd jexec jpub ifconfig sb_jpb1 inet
sb_jpb1: flags=8863<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
        options=8<VLAN_MTU>
        inet 192.168.1.104 netmask 0xffffff00 broadcast 192.168.1.255
# appjail cmd jexec jpub ifconfig sb_jpb2 inet
sb_jpb2: flags=8863<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
        options=8<VLAN_MTU>
        inet 129.0.0.58 netmask 0xffffff00 broadcast 129.0.0.255 
# appjail start jpriv
...
# appjail cmd jexec jpriv ifconfig sb_jpv inet
sb_jpv: flags=8863<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
        options=8<VLAN_MTU>
        inet 129.0.0.88 netmask 0xffffff00 broadcast 129.0.0.255
```
