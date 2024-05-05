Instead of saving space and time, you can copy an entire jail using another jail. It is very simple:

```sh
appjail jail create -I copy=metube metubev2
# recommended:
appjail quick metubev2 \
    copy=metube \
    virtualnet=":<random> default" \
    nat \
    start
```

---

**See also**:

* [Virtual Networks](../networking/virtual-networks/intro.md)
* [LinuxJails](../linux.md)
* [Alias](../networking/alias.md)
* [Alias & Virtual Networks](../networking/virtual-networks/alias-and-virtual-networks.md)
* [NAT](../networking/virtual-networks/NAT.md)
