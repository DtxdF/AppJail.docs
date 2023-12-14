You can probably open a tarball on unix-like and non-unix operating systems, so backing up your jails using this method is very useful.

#### export+jail

##### Syntax

```
output:out_name [portable] [compress:algo]
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

Export the jail directory (`{JAILDIR}/{JAIL_NAME}/jail`).

##### Examples

```sh
appjail jail create -I export+jail="output:nginx.tzst compress:zstd" nginx
```

#### export+root

##### Syntax

```
output:out_name [portable] [compress:algo]
```

###### output

See `export+jail`.

###### portable

Include only portable files. These are the jail directory, the configuration file describing the jail and the initscript.

###### compress

See `export+jail`.

##### Description

Export the root directory of the jail (`{JAILDIR}/{JAIL_NAME}`).

##### Examples

```sh
appjail jail create -I export+root="output:nginx-root.tzst compress:zstd" nginx
```

#### import+jail

##### Syntax

```
input:in_file [portable] [compress:algo]
```

###### input

Tarball to import.

###### portable

Ignored, but used by `import+root`.

###### compress

Ignored, but used by `zfs+import+jail` and `zfs+import+root`.

##### Description

Import the tarball to the jail directory (`{JAILDIR}/{JAIL_NAME}/jail`).

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
input:in_file [portable] [compress:algo]
```

###### input

See `import+jail`.

###### portable

Include only portable files. These are the jail directory, the configuration file describing the jail, the initscript and volumes.

###### compress

See `import+jail`.

##### Description

Import the tarball to the root directory of the jail (`{JAILDIR}/{JAIL_NAME}`).

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
