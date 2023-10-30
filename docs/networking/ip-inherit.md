Inheriting the host network stack is useful for applications that don't need network isolation in any way. However, this will cause some problems for some applications.

To use this network configuration type, just use `ip4_inherit` or `ip6_inherit` or both.

```sh
appjail quick myjail \
    alias \
    ip4_inherit \
    ip6_inherit \
    overwrite \
    start
```
