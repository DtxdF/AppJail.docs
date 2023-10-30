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

**See also**:

* [Virtual Networks](intro.md)
* [LinuxJails](../../linux.md)
