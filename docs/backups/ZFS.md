AppJail can use ZFS to backup a jail in the same way as a tarball file. The great advantage of using this method is that it preserves much more information and is very fast.

!!! warning

    Before using this method, read the [ZFS](../ZFS.md) section to configure it.

## ZFS images

### zfs+export+jail

#### Syntax

```
output:out_name [portable] [compress:algo]
```

##### output

Output name.

##### portable

Ignored, but used by [export+root](#exportroot).

##### compress

It has the same meaning as [export+jail](#exportjail) but does not use `tar(1)`.

#### Description

Recursively export the dataset of the jail (`{ZPOOL}/{ZROOTFS}/{JAIL_NAME}/jail`).

#### Examples

```sh
appjail jail create -I zfs+export+jail="output:myjail.zsnap.gz compress:gzip" myjail
```

### zfs+export+root

#### Syntax

```
output:out_name [portable] [compress:algo]
```

##### output

See [zfs+export+jail](#zfsexportjail).

##### portable

Ignored, but used by [export+root](#exportroot).

##### compress

See [zfs+export+jail](#zfsexportjail).

#### Description

Recursively export the root dataset of the jail (`{ZPOOL}/{ZROOTFS}/{JAIL_NAME}`).

#### Examples

```sh
appjail jail create -I zfs+export+root="output:myjail-root.zsnap.xz compress:xz" myjail
```

### zfs+import+jail

#### Syntax

```
input:in_file [portable] [compress:algo]
```

##### input

The file to import.

##### portable

Ignored, but used by [import+root](#importroot).

##### compress

AppJail can auto-detect the compression used, but you can force any other compression if you wish.

#### Description

Import the file to the dataset of the jail (`{ZPOOL}/{ZROOTFS}/{JAIL_NAME}/jail`).

The root dataset will be created.

#### Examples

##### #1

```sh
appjail jail create -I zfs+import+jail="input:mysql.zsnap.xz" mysql
```

##### #2

```sh
appjail quick mysql \
    zfs+import+jail="input:/tmp/mysql.txz" \
    virtualnet="development:mysql default" \
    nat
```

### zfs+import+root

#### Syntax

```
input:in_file [portable] [compress:algo]
```

##### input

See [zfs+import+jail](#zfsimportjail).

##### portable

Ignored, but used by [import+root](#importroot).

##### compress

See [zfs+import+jail](#zfsimportjail).

#### Description

Import the file to the root dataset of the jail (`{ZPOOL}/{ZROOTFS}/{JAIL_NAME}`).

#### Examples

##### #1

```sh
appjail jail create -I zfs+import+root="input:badwolf.zsnap.zst" badwolf
```

##### #2

```sh
appjail quick apache \
    zfs+import+root="input:/tmp/jweb.txz" \
    bridge="apache iface:jext" \
    dhcp=sb_apache \
    mount_devfs \
    devfs_ruleset=10 \
    start
```

## Clones

Clones is a useful feature of ZFS that saves time and space.

AppJail takes advantage of this feature to clone a jail or a release to create some useful applications. For example, you can clone a generic jail named `webserver` which has its own configuration files, generic files, packages, etc. so you can clone it into a new jail named `nginx`. Another example is making a change to a jail, but copying the whole jail takes a lot of time and space, so cloning here is very useful.

Clones should be used temporarily in the same way as thinjails and you should be aware that you cannot destroy a dataset of a jail on which another jail depends without forcing it (see `appjail jail destroy` for more details).

### clone+jail

#### Syntax

```
jail2clone@snapname
```

##### jail2clone

Jail name to clone.

##### snapname

Snapshot name to be created if it does not exist.

#### Description

Clones a jail and uses it to create another jail.

#### Examples

##### #1

```sh
appjail jail create -I clone+jail=webserver@snap1 nginx
```

##### #2

```sh
appjail quick mariadb clone+jail=jdb@snap1 overwrite start
```

### clone+release

#### Syntax

```
snapname
```

##### snapname

Snapshot name to be created if it does not exist.

#### Description

Clones a release and uses it to create another jail.

Valid types: `thick`, `linux+debootstrap`

#### Examples

##### #1

```sh
appjail jail create -T thick -v 12.3-RELEASE -I clone+release=snap1 bluejail
```

##### #2

```sh
appjail quick bullseye \
    clone+release=linuxsnap1 \
    osversion=bullseye \
    type=linux+debootstrap \
    alias=appjail0 \
    virtualnet="development" \
    linuxfs \
    devfs_ruleset=11 \
    template=/tmp/linux.conf \
    nat=network:development \
    overwrite \
    start
```

---

**See also**:

* [Virtual Networks](../networking/virtual-networks/intro.md)
* [DHCP & SLAAC](../networking/DHCP-and-SLAAC.md)
* [LinuxJails](../linux.md)
* [NAT](../networking/virtual-networks/NAT.md)
