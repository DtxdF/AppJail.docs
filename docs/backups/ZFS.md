AppJail can use ZFS to backup a jail in the same way as a tarball file. The great advantage of using this method is that it preserves much more information and is very fast.

!!! warning

    Before using this method, read the [ZFS](../ZFS.md) section to configure it.

## ZFS images

### zfs+export+jail

#### Syntax

```
output:outname [portable] [compress:algo]
```

##### output

Output name.

##### portable

Ignored, but used by `export+root`.

##### compress

If specified, the file will be compressed.

#### Description

Recursively export the jail dataset to a ZFS image file.

#### Examples

```sh
appjail jail create -I zfs+export+jail="output:myjail.zsnap.gz compress:gzip" myjail
```

### zfs+export+root

#### Syntax

```
output:outname [portable] [compress:algo]
```

##### output

Output name.

##### portable

Ignored, but used by `export+root`.

##### compress

If specified, the file will be compressed.

#### Description

Recursively export the root jail dataset to a ZFS image file.

#### Examples

```sh
appjail jail create -I zfs+export+root="output:myjail-root.zsnap.xz compress:xz" myjail
```

### zfs+import+jail

#### Syntax

```
input:file [portable] [compress:algo]
```

##### input

ZFS image.

##### portable

Ignored, but used by `import+root`.

##### compress

Change the compression algorithm. Automatic detection of the algorithm used by the ZFS image is performed, but if it fails or you need to change for some reason, you do so using this subparameter.

#### Description

Create a new jail by importing a ZFS image into the jail directory.

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
input:file [portable] [compress:algo]
```

##### input

ZFS image.

##### portable

Ignored, but used by `import+root`.

##### compress

Change the compression algorithm. Automatic detection of the algorithm used by the ZFS image is performed, but if it fails or you need to change for some reason, you do so using this subparameter.

#### Description

Create a new jail by importing a ZFS image into the root directory of the jail.

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

Clones should be used temporarily in the same way as thinjails and you should be aware that you cannot destroy a dataset of a jail on which another jail depends without forcing it (see `appjail-jail(1)` `destroy` for more details).

### clone+jail

#### Syntax

```
jail@snapshot
```

##### jail

Jail to create a ZFS snapshot for cloning.

##### snapshot

ZFS snapshot name.

#### Description

Create a new jail by cloning a ZFS <ins>snapshot</ins> of <ins>jail</ins>.

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
snapshot
```

##### snapshot

Snapshot name to be created if it does not exist.

#### Description

Create a new jail by cloning a ZFS <ins>snapshot</ins> of a release.

With this option only the `linux+debootstrap` and `thick` jail types can be used.

#### Examples

##### #1

```sh
appjail jail create -T thick -v 12.3-RELEASE -I clone+release=snap1 bluejail
```

##### #2

```sh
appjail quick bookworm \
    clone+release=linuxsnap1 \
    osversion=bookworm \
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
