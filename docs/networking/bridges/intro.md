AppJail can create bridges and epairs on demand using this option. This is very useful to provide connection with the outside to some jails when it is really necessary.

This provides isolation that NAT cannot, so combining NAT and this option does not make much sense.

```sh
appjail quick jbridge \
    bridge="jpub iface:jext" \
    start
# custom bridge
appjail quick jbridge \
    bridge="jpub iface:jext bridge:public" \
    start
```

AppJail will create two interfaces named `s[ab]_jpub`. The `sa_jpub` interface is attached to the bridge and the `sb_jpub` is used by the jail.

AppJail does not create bridges and epairs unless they do not exist. It also cannot add an interface as a member of a bridge when it is already added.

By default, a bridge named `SHARED_BRIDGE` defined in your AppJail configuration file, is created unless you provide another name as you have seen.

Suppose we are installing packages and we don't want to provide connection to the outside until we really need it. AppJail can detach an interface that is a member of a bridge using `appjail-network(1)` `detach`.

```sh
appjail network detach jpub
```

However, it is necessary to edit the template using `appjail-config edit -j jbridge` and remove the lines where AppJail attaches the interface to not provide connection to the outside when the jail is restarted.

AppJail does not destroy an interface that is not a member of the specified bridge, so if we stop the jail using `appjail-stop(1)` the `s[ab]_jpub` interface is still in the system. To force the destruction of an `if_epair(4)` interface use `appjail-network(1)` `detach` `-df`.

```sh
appjail network detach -df jpub
```

!!! warning
    
    An interface cannot be used on a bridge that is a member of another bridge.

!!! warning

    If you use bridges on a host with a packet filter, you should set at least `net.link.bridge.pfil_member` and `net.link.bridge.pfil_bridge`, and depending on your needs, `net.link.bridge.pfil_onlyip` to `0`. Read `if_bridge(4)` for an explanation of these sysctls. If you don't do this you will have no connection with the outside.

    !!! warning

        For this to take effect, you need to load the following modules into your `loader.conf(5)`:

        ```
        if_bridge_load="YES"
        bridgestp_load="YES"
        ```

---

**See also**:

* [Templates](../../templates.md)
