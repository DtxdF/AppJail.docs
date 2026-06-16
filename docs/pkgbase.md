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

## quarterly vs latest

By default, AppJail uses the `latest` branch to bootstrap the files using `pkgbase(8)`, but AppJail includes a sample configuration file, `pkg.conf(5)`, that uses `quarterly` instead.

```console
# cat /usr/local/share/appjail/files/pkgbase/base.conf 
FreeBSD-base: {
  url: "https://pkg.FreeBSD.org/${ABI}/base_latest",
  mirror_type: "srv",
  signature_type: "fingerprints",
  fingerprints: "/usr/share/keys/pkg",
  enabled: yes
}
# cat /usr/local/share/examples/appjail/pkgbase/quarterly-release/base.conf 
FreeBSD-base: {
  url: "pkg+https://pkg.FreeBSD.org/${ABI}/base_release_${VERSION_MINOR}",
  mirror_type: "srv",
  signature_type: "fingerprints",
  fingerprints: "/usr/share/keys/pkgbase-${VERSION_MAJOR}",
  enabled: yes
}
```

Generally, jail managers only restrict the creation of release directories based on architecture and version. In AppJail, this restriction is extended to include the release name, so you can have different releases with the same version number and architecture but with subtle differences. In our case, we can take advantage of this to create a release directory using the `quarterly` branch.

```console
# appjail fetch pkgbase -v 15 -r quarterly -c /usr/local/share/examples/appjail/pkgbase/quarterly-release -f /usr/share/keys/pkgbase-15
...
# appjail update release quarterly
...
# appjail fetch list amd64/15
NAME
default
quarterly
```

If this is the first time you're creating the release directory, you don't need to assign it a name (don't configure it with `-r`), and AppJail will use the default one; however, I recommend that you have one release directory for `latest` (in my case this is `amd64/15/default`) and another for `quarterly`, so you can easily create jails using different branches when necessary.

```console
# appjail quick jlatest start overwrite=force alias ip4_inherit ephemeral
...
# appjail cmd jexec jlatest cat /etc/pkg/FreeBSD.conf
#
# To disable a repository, instead of modifying or removing this file,
# create a /usr/local/etc/pkg/repos/FreeBSD.conf file, e.g.:
#
#   mkdir -p /usr/local/etc/pkg/repos
#   echo "FreeBSD-ports: { enabled: no }" > /usr/local/etc/pkg/repos/FreeBSD.conf
#   echo "FreeBSD-ports-kmods: { enabled: no }" >> /usr/local/etc/pkg/repos/FreeBSD.conf
#
# Note that the FreeBSD-base repository is disabled by default.
#

FreeBSD-ports: {
  url: "pkg+https://pkg.FreeBSD.org/${ABI}/latest",
  mirror_type: "srv",
  signature_type: "fingerprints",
  fingerprints: "/usr/share/keys/pkg",
  enabled: yes
}
FreeBSD-ports-kmods: {
  url: "pkg+https://pkg.FreeBSD.org/${ABI}/kmods_latest",
  mirror_type: "srv",
  signature_type: "fingerprints",
  fingerprints: "/usr/share/keys/pkg",
  enabled: yes
}
FreeBSD-base: {
  url: "pkg+https://pkg.FreeBSD.org/${ABI}/base_latest",
  mirror_type: "srv",
  signature_type: "fingerprints",
  fingerprints: "/usr/share/keys/pkg",
  enabled: no
}
# appjail quick jquarterly start overwrite=force alias ip4_inherit release=quarterly ephemeral
...
# appjail cmd jexec jquarterly cat /etc/pkg/FreeBSD.conf
#
# To disable a repository, instead of modifying or removing this file,
# create a /usr/local/etc/pkg/repos/FreeBSD.conf file, e.g.:
#
#   mkdir -p /usr/local/etc/pkg/repos
#   echo "FreeBSD-ports: { enabled: no }" > /usr/local/etc/pkg/repos/FreeBSD.conf
#   echo "FreeBSD-ports-kmods: { enabled: no }" >> /usr/local/etc/pkg/repos/FreeBSD.conf
#
# Note that the FreeBSD-base repository is disabled by default.
#

FreeBSD-ports: {
  url: "pkg+https://pkg.FreeBSD.org/${ABI}/quarterly",
  mirror_type: "srv",
  signature_type: "fingerprints",
  fingerprints: "/usr/share/keys/pkg",
  enabled: yes
}
FreeBSD-ports-kmods: {
  url: "pkg+https://pkg.FreeBSD.org/${ABI}/kmods_quarterly_${VERSION_MINOR}",
  mirror_type: "srv",
  signature_type: "fingerprints",
  fingerprints: "/usr/share/keys/pkg",
  enabled: yes
}
FreeBSD-base: {
  url: "pkg+https://pkg.FreeBSD.org/${ABI}/base_release_${VERSION_MINOR}",
  mirror_type: "srv",
  signature_type: "fingerprints",
  fingerprints: "/usr/share/keys/pkgbase-${VERSION_MAJOR}",
  enabled: no
}
```

---

**See also**:

* [Virtual Networks](networking/virtual-networks/intro.md)
* [NAT](networking/virtual-networks/NAT.md)
* [Update & Upgrade](update-and-upgrade.md)
