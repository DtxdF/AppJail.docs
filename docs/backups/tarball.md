You can probably open a tarball on unix-like and non-unix operating systems, so backing up your jails using this method is very useful.

#### export+jail

##### Syntax

```
output:outname [portable] [compress:algo]
```

###### output

Tarball name.

###### portable

Ignored, but used by `export+root`.

###### compress

Compress the tarball using the following methods:

* `bzip`
* `gzip`
* `lrzip`: requires `archivers/lrzip`.
* `lz4`.
* `lzma`
* `lzop`: requires `archivers/lzop`.
* `xz`
* `zstd`

If `compress` is not defined, no compression is applied.

##### Description

Export the jail directory to a tarball file.

##### Examples

```sh
appjail jail create -I export+jail="output:nginx.tzst compress:zstd" nginx
```

#### export+root

##### Syntax

```
output:outname [portable] [compress:algo]
```

###### output

Output name.

###### portable

Include only portable files, that is, the jail directory, the **InitScript**, the configuration file that describes the jail, and the specifications of volumes used by the jail. This is used by `appjail-image(1)`.

###### compress

If specified, the file will be compressed.

##### Description

Export the root directory of the jail to a tarball file.

##### Examples

```sh
appjail jail create -I export+root="output:nginx-root.tzst compress:zstd" nginx
```

#### import+jail

##### Syntax

```
input:file [portable] [compress:algo]
```

###### input

Tarball file.

###### portable

Ignored, but used by `import+root`.

###### compress

Ignored, but used by `zfs+import+jail` and `zfs+import+root`.

##### Description

Create a new jail by importing a tarball file into the jail directory.

##### Examples

###### #1

```sh
appjail jail create -I import+jail="input:nginx.tzst" othernginx
```

###### #2

```sh
appjail quick nginx \
    import+jail="input:/tmp/web3.txz" \
    virtualnet="development:nginx default" \
    nat \
    expose=8082:80 \
    limits=vmemoryuse:deny=512m \
    start
```

#### import+root

##### Syntax

```
input:file [portable] [compress:algo]
```

###### input

Tarball file.

###### portable

Include only portable files, that is, the jail directory, the **InitScript**, the configuration file that describes the jail, and the specifications of volumes used by the jail. This is used by `appjail-image(1)`.

###### compress

Ignored, but used by `zfs+import+root` and `zfs+import+root`.

##### Description

Create a new jail by importing a tarball file into the root directory of the jail.

##### Examples

###### #1

```sh
appjail jail create -I import+root="input:nginx.tzst" nginx
```

###### #2

```sh
appjail quick jweb \
    import+root="input:/tmp/web3.tgz" \
    vnet=em0 \
    dhcp=em0 \
    mount_devfs \
    devfs_ruleset=10 \
    overwrite \
    start
```

---

**See also**:

* [Virtual Networks](../networking/virtual-networks/intro.md)
* [Resource limits (RACCT/RCTL)](../limits.md)
* [NAT](../networking/virtual-networks/NAT.md)
* [Port Forwarding](../networking/virtual-networks/port-forwarding.md)
* [DHCP & SLAAC](../networking/DHCP-and-SLAAC.md)
* [VNET](../networking/VNET.md)
