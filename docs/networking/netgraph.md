The `jng` script uses some Netgraph nodes, such as `ng_bridge(4)` and `ng_eiface(4)` to attach an interface. AppJail can use this script to get the advantages of using Netgraph in jails.

AppJail will load the following kernel module before using this option:

* `ng_ether`

```sh
appjail quick myjail \
    jng="myjail jext" \
    overwrite \
    start
```

`myjail` is the name used for the links and `jext` is your interface that will be attached to the bridge. In the above example, `jng` will create a node named `ng0_myjail` and a bridge named `jextbridge`.

!!! warning

    You need to install the `jng` script before using this option. Run
    `install -m 555 /usr/share/examples/jails/jng /usr/local/bin/jng`
    to install it.
