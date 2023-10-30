IPv4 or IPv6 can be disabled in AppJail with the `ip4_disable` or `ip6_disable` options.

```sh
appjail quick myjail \
    alias \
    ip4_disable \
    ip6_disable \
    overwrite \
    start
```
