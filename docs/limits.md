AppJail can limit jail resources using `rctl(4)`. To use it, you need to enable it in `loader.conf(5)` and reboot your system.

```
kern.racct.enable=1
```

Like many AppJail commands, limits can be set using `appjail quick` or its own command to apply `rctl(4)` rules to an existing jail.

```sh
# Using appjail quick:
appjail quick nginx \
    virtualnet="web:nginx default" \
    nat \
    expose=80 \
    limits="vmemoryuse:deny=512m" \
    limits="vmemoryuse:log=450m" \
    limits="maxproc:log=30" \
    start \
    overwrite

# Manually:
appjail quick nginx virtualnet="web:nginx default" nat expose=80 overwrite
appjail limits set nginx vmemoryuse:deny=512m
appjail limits set nginx vmemoryuse:log=450m
appjail limits set nginx maxproc:log=30
appjail-config set -Ij nginx exec.created='appjail limits on ${name}'
appjail-config set -Ij nginx exec.created='appjail limits off ${name}'
appjail start nginx
```

To display the current rules of a jail, run `appjail limits list`.

```console
# appjail limits list nginx
NRO  ENABLED  NAME  RULE                  LOADED
0    1        -     vmemoryuse:deny=512m  jail:nginx:vmemoryuse:deny=512M
1    1        -     vmemoryuse:log=450m   jail:nginx:vmemoryuse:log=450M
2    1        -     maxproc:log=30        jail:nginx:maxproc:log=30
```

In addition, we can use AppJail to display resource usage in a table-like interface.

```console
# appjail limits stats myjail
MAXPROC  CPUTIME  PCPU  VMEMORYUSE  READIOPS  WRITEIOPS
7        13       0     99M         0         0
```

AppJail only shows a few keywords by default, but you can get all keywords defined in `rctl(8)`.

```console
# appjail limits stats myjail openfiles cputime datasize stacksize
OPENFILES  CPUTIME  DATASIZE  STACKSIZE
1576       13       1008K     0
```

**See also**:

* [Virtual Networks](networking/virtual-networks/intro.md)
* [NAT](networking/virtual-networks/NAT.md)
* [Port Forwarding](networking/virtual-networks/port-forwarding.md)
* [Templates](templates.md)
