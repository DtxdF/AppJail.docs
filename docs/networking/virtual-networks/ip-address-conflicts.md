There are some problems when using the installation methods described above and when using virtual networks: duplicate addresses and jails with the same network name but with different network address.

!!! warning

    The problems described here are when using `appjail jail` to create a jail using
    the installation methods described in previous sections, but don't worry when
    using `appjail quick` because this command forcibly reserves an IP address.

```sh
appjail quick origin \
    virtualnet="db:origin default" \
    nat \
    mount_devfs \
    start \
    overwrite
appjail jail create -I clone+jail=origin@snap1 dup
```

`appjail jail list` shows the problem:

```console
# appjail jail list | grep -Ee '(origin|dup)'
DOWN    dup          thin  13.1-RELEASE  -       10.42.0.4
UP      origin       thin  13.1-RELEASE  -       10.42.0.4
```

If you want to know the total number of duplicate IPs in a network, use `appjail network hosts -d`:

```console
# appjail network hosts -dn db
   2 10.42.0.4
```

To solves this problem, use `appjail network reserve` and use `-a forceauto` to get an available IP from the pool. You can use a specific IP address and AppJail will check if it is in a valid range and another jail does not use it. Do not use `auto` because it does not reserve an IP address when the jail already has one.

```console
# appjail network reserve -j dup -n web -a forceauto
10.42.0.5
```

The problem has been solved:

```console
# appjail jail list | grep -Ee '(origin|dup)'
DOWN    dup          thin  13.1-RELEASE  -       10.42.0.5
UP      origin       thin  13.1-RELEASE  -       10.42.0.4
```

!!! tip
    
    A much easier way to solve the above problem is to use the `appjail network fix dup` command.

Another problem that can occur is when importing a jail that has a network with the same name as an existing one on the host, but with a different IP range.

```console
# appjail jail list
DOWN    jtest        thin               13.1-RELEASE  -      172.0.0.2
DOWN    badwolf      thin               13.1-RELEASE  -      10.32.0.11
DOWN    php          thin               13.1-RELEASE  -      10.32.0.9
DOWN    mariadb      thin               13.1-RELEASE  -      10.32.0.10
DOWN    nginx        thin               13.1-RELEASE  -      10.32.0.16
DOWN    teleirc      thin               13.1-RELEASE  -      10.32.0.6
```

The above example reveals a big problem: `jtest` has a correct IP address, but `badwolf`, `php`, `mariadb`, `nginx` and `teleirc` have an invalid range since they were imported from another host. You can use the instructions explained earlier in the section, but AppJail has a useful command that simplifies this problem: `appjail network fix addr`.

```console
# appjail network fix addr
[00:00:00] [ debug ] Fixing IPv4 addresses that are not in the development network ...
[00:00:03] [ debug ] Fixed: jail:teleirc, old:10.32.0.6, new:172.0.0.3
[00:00:05] [ debug ] Fixed: jail:php, old:10.32.0.9, new:172.0.0.4
[00:00:08] [ debug ] Fixed: jail:mariadb, old:10.32.0.10, new:172.0.0.5
[00:00:11] [ debug ] Fixed: jail:badwolf, old:10.32.0.11, new:172.0.0.6
[00:00:13] [ debug ] Fixed: jail:nginx, old:10.32.0.16, new:172.0.0.7
```

---

**See also**:

* [Virtual Networks](intro.md)
* [NAT](NAT.md)
* [Backups](../../backups/intro.md)
