PkgBase is a system that allows the base system of FreeBSD to be managed with the `pkg(8)` package management tool, just like third-party software in the ports and packages collection.

PkgBase replaces:

* `.txz` distribution sets, which are used for installation of the OS with `bsdinstall(8)`
* `freebsd-update(8)` for updates to the OS. 

PkgBase complements building and installing from source. In particular:

* Simple installations of FreeBSD-CURRENT and FreeBSD-STABLE can be updated with pkg - there's no longer a requirement to build from source. 

In the world of FreeBSD jails, PkgBase stands out, so how can we create jails using AppJail with PkgBase? As with distribution sets, `appjail-fetch(1)` is the first character to appear on the scene.

```console
# appjail fetch pkgbase
...
# appjail fetch list
appjail fetch list
ARCH   VERSION  NAME
amd64  14       default
```

When no major version of FreeBSD is specified, the above command detects it and creates the release anyway. Unlike the `www` method of `appjail-fetch(1)`, this subcommand only works with major versions, since we are constructing (internally) the ABI string, which in my case is `FreeBSD:14:amd64`, where `amd64` is the architecture (another parameter that this command automatically detects when none is specified).

When no package is specified in the positional arguments, this subcommand performs black magic. If executed from `FreeBSD:14:*`, a bunch of packages are installed that are best described in `appjail-fetch(1)`, but for versions higher than that (e.g.: `FreeBSD:15:*`), only `FreeBSD-set-minimal-jail` is installed, which is a metapackage that installs a set of packages sufficient for most use cases. PkgBase is not available in versions such as `FreeBSD:13:*` or lower.

```console
# appjail quick j1 \
    ephemeral \
    overwrite=force \
    osversion=14 \
    virtualnet=":<random> default" \
    nat \
    start
...
# appjail jail list -j j1
STATUS  NAME  ALT_NAME  TYPE  VERSION  PORTS  NETWORK_IP4
UP      j1    -         thin  14       -      10.0.0.6
```

In general, the workflow remains unchanged: the release is created is not available, the jail is created with that release, and the jail is used. However, as can be seen above, it is implicitly the type of jail, which by default is a thin jail, but with PkgBase, thick jails may be a good choice.

```console
# appjail quick j1 \
    ephemeral \
    overwrite=force \
    osversion=14 \
    virtualnet=":<random> default" \
    nat \
    start \
    type=thick
...
# appjail jail list -j j1
STATUS  NAME  ALT_NAME  TYPE   VERSION  PORTS  NETWORK_IP4
UP      j1    -         thick  14       -      10.0.0.6
```

And since `appjail-update(1)` only works with thick jails, you can use it to perform minor updates.

```console
# appjail update release -v 14
...
# appjail update jail j1
```

AppJail detects that you have used PkgBase previously, so it uses `pkg(8)` instead of `freebsd-update(8)`.

---

**See also**:

* [Virtual Networks](networking/virtual-networks/intro.md)
* [NAT](networking/virtual-networks/NAT.md)
* [Update & Upgrade](update-and-upgrade.md)
