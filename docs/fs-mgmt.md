AppJail uses its own way to use the `fstab(5)` file if you don't use `mount.fstab` as you can see in `Templates`.

The main motivation is to allow mounting devices with strange characters, like a space in a directory. Especially in shared environments when users have no control over how they can name their files and directories. `fstab(5)` allows you to mount such devices, but it is not automated, so `appjail fstab` does it for you.

Another reason to get a specific order of the fields used by a jail and a useful description that allows you or another sysadmin to remember them very easily.

`appjail fstab` has some useful commands. For example, you can list the current entries using `appjail fstab jail <jail name> list`.

```console
# appjail fstab jail debian list
NRO  ENABLED  NAME  DEVICE     MOUNTPOINT  TYPE       OPTIONS               DUMP  PASS
0    1        -     devfs      /dev        devfs      rw,ruleset=0          0     0
1    1        -     tmpfs      /dev/shm    tmpfs      rw,size=1g,mode=1777  0     0
2    1        -     fdescfs    /dev/fd     fdescfs    rw,linrdlnk           0     0
3    1        -     linprocfs  /proc       linprocfs  rw                    0     0
4    1        -     linsysfs   /sys        linsysfs   rw                    0     0
```

It shows us their fields and their meanings are:

* `NRO`: NRO is an arbitrary number representing the entry and order in which `appjail fstab jail ... compile` writes to the resulting `fstab(5)`. AppJail generates the NRO using the last NRO plus 1 when the `-n` parameter is not used in `appjail fstab jail ... set`or `0` is used when there is no entry.
* `ENABLED`: Indicates whether this entry will be written to the resulting `fstab(5)` file.
* `DEVICE`, `MOUNTPOINT`, `TYPE`, `OPTIONS`, `DUMP`, `PASS`: Represents the entries in the `fstab(5)` file.

We can add more entries using the `appjail fstab jail ... set` command. For example, to run x11 applications we need to mount `/tmp/.X11-unix`.

```console
# appjail cmd local debian mkdir -p tmp/.X11-unix
# appjail fstab jail debian set -d /tmp/.X11-unix -m /tmp/.X11-unix
# appjail fstab jail debian list
NRO  ENABLED  NAME  DEVICE          MOUNTPOINT      TYPE       OPTIONS               DUMP  PASS
0    1        -     devfs           /dev            devfs      rw,ruleset=0          0     0
1    1        -     tmpfs           /dev/shm        tmpfs      rw,size=1g,mode=1777  0     0
2    1        -     fdescfs         /dev/fd         fdescfs    rw,linrdlnk           0     0
3    1        -     linprocfs       /proc           linprocfs  rw                    0     0
4    1        -     linsysfs        /sys            linsysfs   rw                    0     0
5    1        -     /tmp/.X11-unix  /tmp/.X11-unix  nullfs     rw                    0     0
```

If you want to change a field of an existing entry, just specify the NRO and the corresponding parameter. `-d` and `-m` are not required when defined. For example, to change the `mount(8)` options in NRO `0`:

```console
appjail fstab jail debian set -n 0 -o rw,ruleset=11
```

To mount those device we can either restart the jail or use `appjail fstab jail ... compile` and `appjail fstab jail ... mount -a`.

```sh
appjail restart debian
# or
appjail fstab jail debian compile
appjail fstab jail debian mount -a
```

`appjail fstab` can also see the mounted devices for the given jail. For example, I want to see the mounted devices for my jail named `movies`:

```console
# appjail fstab jail movies set -d /dev/da0s1 -m /mnt/07181 -t msdosfs
# appjail fstab jail movies list
NRO  ENABLED  NAME  DEVICE      MOUNTPOINT  TYPE     OPTIONS  DUMP  PASS
0    1        -     /dev/da0s1  /mnt/07181  msdosfs  rw       0     0
# appjail fstab jail movies mounted
/usr/local/appjail/releases/amd64/13.1-RELEASE/default/release -> /usr/local/appjail/jails/movies/jail/.appjail
/dev/da0s1 -> /usr/local/appjail/jails/movies/jail/mnt/07181
devfs -> /usr/local/appjail/jails/movies/jail/dev
```

That device is not very descriptive. I want to give it a name that represents its contents.

```console
# appjail fstab jail movies set -n 0 -N 'The Godfather Trilogy'
# appjail fstab jail movies list -n 0
NRO  ENABLED  NAME                   DEVICE      MOUNTPOINT  TYPE     OPTIONS  DUMP  PASS
0    1        The Godfather Trilogy  /dev/da0s1  /mnt/07181  msdosfs  rw       0     0
```

Since it is a USB, the jail cannot be gracefully stopped or it will not boot because the device is gone. To solve this I will run the following commands:

```console
# appjail fstab jail movies umount mnt/07181
# appjail fstab jail movies set -n 0 -E
# appjail fstab jail movies compile
# appjail fstab jail movies list -n 0
NRO  ENABLED  NAME                   DEVICE      MOUNTPOINT  TYPE     OPTIONS  DUMP  PASS
0    0        The Godfather Trilogy  /dev/da0s1  /mnt/07181  msdosfs  rw       0     0
```

The jail can now be stopped without any problems.

`appjail fstab` has the `fstab` option to mount devices in the jail. Let's create a jail to compile ports:

```console
# appjail quick jcomp fstab="/usr/ports /usr/ports" virtualnet=":<random> default" nat start
...
# appjail fstab jail jcomp list
NRO  ENABLED  NAME  DEVICE      MOUNTPOINT  TYPE    OPTIONS  DUMP  PASS
0    1        -     /usr/ports  /usr/ports  nullfs  rw       0     0
```

### PseudoFS

An interesting and useful feature of `appjail fstab` is when you set `type` to `<pseudofs>`. It is actually a pseudo-filesystem as the name implies, in other words, this does not exist on your system.

The purpose of this handy feature is to allow you to easily separate the data that should persist when removing the jail. For example, imagine you import an image and it comes with `/usr/local/www/apache24/data/wp-content` indicating a WordPress installation. Such files and subdirectories will be removed with the jail data and other things it contains. For this data to persist, you must move them to the host and mount them using `mount_nullfs(8)`. You will probably need to stop the jail before moving the files as some applications may not be able to run correctly.

This pseudo-filesystem does this. It moves the data from the jail to the host when you run `appjail fstab jail ... compile` and mounts that file or directory using `mount_nullfs(8)`, so that when you remove the jail, your data is safe.

As a side note, `PASS` and `DUMP` will be ignored.

Using `<pseudofs>` is no different than using another file system.

```console
# mkdir -p /tmp/var_tmp
# ls /tmp/var_tmp
# appjail fstab jail jtest set -d /tmp/var_tmp -m /var/tmp -t '<pseudofs>'
# appjail fstab jail jtest
NRO  ENABLED  NAME  DEVICE        MOUNTPOINT  TYPE        OPTIONS  DUMP  PASS
0    1        -     /tmp/var_tmp  /var/tmp    <pseudofs>  rw       0     0
# appjail restart jtest
...
[00:00:10] [ debug ] [jtest] Moving /usr/local/appjail/jails/jtest/jail//var/tmp/vi.recover -> /tmp/var_tmp/vi.recover ...
...
# appjail fstab jail jtest mounted
/usr/local/appjail/releases/amd64/13.2-RELEASE/default/release -> /usr/local/appjail/jails/jtest/jail/.appjail
/tmp/var_tmp -> /usr/local/appjail/jails/jtest/jail/var/tmp
devfs -> /usr/local/appjail/jails/jtest/jail/dev
# ls /tmp/var_tmp
vi.recover/
```

It is preferable and advisable to reboot, since as mentioned above, an application may have problems when moving files and directories here and there while running.

### Notes

`appjail fstab jail ... compile` will do some things for you. If the file system type is `nullfs` it will create the file or directory inside the jail specified by `MOUNTPOINT` depending on whether `DEVICE` is a file or directory, or an error is displayed when the file's type is not a file or directory. If the file system type is `<pseudofs>` it will perform the same things as `nullfs` plus other things described in `PseudoFS`. If none of these file system types match and `MOUNTPOINT` does not exist inside the jail, a directory pointing to that path will be created.

---

**See also**:

* [Virtual Networks](networking/virtual-networks/intro.md)
* [LinuxJail](linux.md)
