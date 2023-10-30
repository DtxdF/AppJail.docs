AppJail can use DHCP or SLAAC to rely on another node in the network. As you can see in the previous sections each option has its own way to rename an interface, but fortunately it is easy to remember.

Before using DHCP, we need to configure a ruleset in our `devfs.rules(5)` to allow `bpf(4)` and other relevant devices.

**/etc/devfs.rules**:

```
[devfsrules_dhcp=10]
add include $devfsrules_jail_vnet
add path 'bpf*' unhide
```

```sh
# VNET + DHCP
 appjail quick jdhcp \
    vnet=em0 \
    dhcp=em0 \
    mount_devfs \
    devfs_ruleset=10 \
    start
# Netgraph + DHCP
appjail quick jdhcp \
    jng="jdhcp em0" \
    dhcp=ng0_jdhcp \
    mount_devfs \
    devfs_ruleset=10 \
    start \
    overwrite
# Netgraph + Bridge + DHCP
appjail quick jdhcp \
    jng="jdhcp em0" \
    bridge="iface:jext jdhcp" \
    dhcp=ng0_jdhcp \
    dhcp=sb_jdhcp \
    mount_devfs \
    devfs_ruleset=10 \
    start \
    overwrite
```

IPv6 has grown rapidly in some countries and one way to configure an interface that wants to use this new version of IP is to rely on SLAAC. You can use AppJail to create a jail that use SLAAC in the same way as DHCP.

```sh
# VNET + SLAAC
appjail quick jslaac \
    vnet="em0" \
    slaac="em0" \
    start \
    overwrite
# VNET + SLAAC + Bridge + DHCP
appjail quick jds \
    vnet="em0" \
    slaac="em0" \
    bridge="iface:jext jds" \
    dhcp="sb_jds" \
    mount_devfs \
    devfs_ruleset=10 \
    start
```

!!! warning

    Remember that any interface you use with VNET disappears, so you will probably
    want to stop the jail using that interface if you want to use it on another.

!!! tip

    I recommend reading `Pragmatic IPv6 (part 1)` and `Pragmatic IPv6 (part 2)` by `Hiroki Sato`.
