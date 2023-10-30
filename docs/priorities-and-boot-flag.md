AppJail can start jails at system startup using the `appjail` RC script which is just a wrapper for `appjail startup`. But this command only starts jails that have the `boot flag` set to `on` and the order can be specified using priorities.

!!! tip

    It is not necessary to set the startup flag explicitly, as it is already set
    by default.

Since these features are very useful when used in conjunction with [Dependent Jails](dependent-jails.md) we will use the jail used in that section.

```sh
appjail jail boot on nginx
```

We can use `appjail jail list` to see if the `boot flag` is enabled for this jail.

```console
# appjail jail list -j nginx boot name
BOOT  NAME
1     nginx
```

`appjail quick` can create a jail with the `boot flag`.

```sh
appjail quick myjail boot overwrite
```

Jails have the same priority at the time of their creation (unless you use the `priority` option in `appjail quick`), so `appjail startup` will start the jails in the order they appear. If we want to start a jail before the others we have to change the priority using `appjail jail priority`.

```sh
appjail jail priority -p 10 myjail
```

The `nginx` jail will be started first because `0` is high priority.

Let's see the current priorities of our jails:

```console
# appjail jail list boot priority name
BOOT  PRIORITY  NAME
0     0         mariadb
1     10        myjail
1     0         nginx
0     0         php-fpm
0     0         vjail
```

`appjail startup` will stop the previous jails in reverse order.

!!! note "Notes"

    1. By default, when a jail is created using `appjail quick` the boot flag is enabled by that jail. See AppJail configuration file for more details.
    2. If `USE_PARALLEL` is enabled (default: `1`), AppJail starts the jails in parallel with the same priority in that order.
