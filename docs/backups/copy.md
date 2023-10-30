Instead of saving space and time, you can copy an entire jail using another jail. It is very simple:

```sh
appjail jail create -I copy=bullseye debian11
# or
appjail quick debian11 \
    copy=bullseye \
    alias=appjail0 \
    virtualnet="development" \
    devfs_ruleset=11 \
    template=/tmp/linux.conf \
    nat=network:development \
    start
```

---

**See also**:

* [Virtual Networks](../networking/virtual-networks/intro.md)
* [LinuxJails](../linux.md)
* [Alias](../networking/alias.md)
* [Alias & Virtual Networks](../networking/virtual-networks/alias-and-virtual-networks.md)
* [NAT](../networking/virtual-networks/NAT.md)
