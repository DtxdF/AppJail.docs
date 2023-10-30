AppJail can use IP aliases in a jail. This type of configuration does not provide isolation, but may be useful for some applications.

**IPv4**:

```sh
appjail quick myjail \
    alias=jext \
    ip4="192.168.1.120/24" \
    overwrite \
    start
```

**IPv6**:

```sh
appjail quick myjail \
    alias=jext \
    ip6="2001:db8:0:1::2/64" \
    overwrite \
    start
```

**Dual**:

```sh
appjail quick myjail \
    alias=em0 \
    ip4="192.168.1.120/24" \
    ip6="2001:db8:0:1::2/64" \
    overwrite \
    start
```

**Multiple Interfaces**:

```sh
appjail quick myjail \
    alias \
    ip4="jext|192.168.1.120/24" \
    ip6="em0|2001:db8:0:1::2/64" \
    overwrite \
    start
```

!!! info
    
    The extra option `overwrite` is to stop and destroy the jail if it exists.
