A linux distro in a jail is very useful for running isolated applications that are not currently in the FreeBSD ports collection.

AppJail will load the following kernel modules before using this option:

* `fdescfs`
* `linprocfs`
* `tmpfs`
* `linsysfs`
* `pty`
* `linux`: `amd64` and `i386` only.
* `linux64`: `amd64` and `aarch64` only.

```sh
appjail fetch debootstrap bookworm
appjail quick debian \
    osversion=bookworm \
    type=linux+debootstrap \
    start \
    linuxfs \
    devfs_ruleset=0 \
    template=/tmp/linux.conf \
    login
```

`appjail-fetch(1)` will download Debian Bullseye. We need to set the release version with `osversion`, use the `linuxfs` option to mount the filesystems used by the linux distribution, use the ruleset `0` and use a linux-specific template.

The ruleset can be different, you just need to allow `/dev/shm` and `/dev/fd`.

The linux template is as follows:

```
exec.start: "/bin/true"
exec.stop: "/bin/true"
persist
```

!!! warning

    You need to install `sysutils/debootstrap` before using this method.
