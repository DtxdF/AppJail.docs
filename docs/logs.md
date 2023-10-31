AppJail has its own way of saving logs. Fortunately, it is not difficult to learn.

AppJail uses four things: `type`, `entity`, `subtype` and `log`.

* `type`: refers to a group of entities with the same meaning in a context.
* `entity`: refers to an individual thin in a group.
* `subtype`: refers to a group of logs with the same mearing in a context.
* `log`: name of the log.

**types**:

* `commands`: Logs created by commands.
* `jails`: Logs created by jails.
* `nat`: Logs created by the `appjail nat` command or something similar.

**entity**

The entities can be a network, a jail or something similar. They are dynamic.

**subtype**:

* `jails/{ENTITY}/build`: Output of a compilation when executing `update jail`. This is for thickjails that are installed using a source tree.
* `jails/{ENTITY}/console`: A file to direct command output (stdout and stderr) to.
* `jails/{ENTITY}/healthcheckers`: Where healthcheckers logs their output.
* `{jails|nat}/{ENTITY}/{startup-start|startup-stop}`: Logs created by the `appjail startup` command.
* `releases/{ENTITY}/build`: Output of a compilation when executing `fetch src`.
* `commands/{ENTITY}/output`: When `ENABLE_LOGGING_OUTPUT` is set to 1, AppJail will log all output of a command.
* `{jails|releases}/${ENTITY}/etcupdate`: Output of `etcupdate(8)` when executing `etcupdate`.

**log**:

The name of the logs can be changed as desired. See the configuration file for details.

Logs can be listed simply using `appjail logs` with no arguments or executing `appjail logs list`.

```console
# appjail logs
TYPE   ENTITY     SUBTYPE        LOG
jails  debian     console        2023-02-03.log
jails  debian     console        2023-02-04.log
jails  debian     startup-start  2023-02-03.log
jails  debian     startup-start  2023-02-04.log
jails  debian     startup-stop   2023-02-03.log
jails  debian     startup-stop   2023-02-04.log
jails  jalias     console        2023-02-03.log
jails  jalias     console        2023-02-04.log
jails  jalias     startup-start  2023-02-03.log
jails  jalias     startup-start  2023-02-04.log
jails  jalias     startup-stop   2023-02-03.log
jails  jalias     startup-stop   2023-02-04.log
jails  jalias46   console        2023-02-03.log
jails  jalias46   console        2023-02-04.log
jails  jalias46   startup-start  2023-02-03.log
jails  jalias46   startup-start  2023-02-04.log
jails  jalias46   startup-stop   2023-02-03.log
jails  jalias46   startup-stop   2023-02-04.log
jails  jalias6    console        2023-02-03.log
jails  jalias6    console        2023-02-04.log
jails  jalias6    startup-start  2023-02-03.log
jails  jalias6    startup-start  2023-02-04.log
jails  jalias6    startup-stop   2023-02-03.log
jails  jalias6    startup-stop   2023-02-04.log
jails  jbridge    console        2023-02-03.log
jails  jbridge    console        2023-02-04.log
jails  jbridge    startup-start  2023-02-04.log
jails  jbridge    startup-stop   2023-02-04.log
jails  jdb        console        2023-02-03.log
jails  jdb        console        2023-02-04.log
jails  jdb        startup-start  2023-02-04.log
jails  jdb        startup-stop   2023-02-03.log
jails  jdb        startup-stop   2023-02-04.log
jails  jdev       console        2023-02-03.log
jails  jdev       console        2023-02-04.log
jails  jdev       startup-start  2023-02-04.log
jails  jdev       startup-stop   2023-02-04.log
jails  jdhcp      console        2023-02-03.log
jails  jdhcp      startup-start  2023-02-04.log
jails  jdhcp      startup-stop   2023-02-03.log
jails  jdisable   console        2023-02-03.log
jails  jdisable   console        2023-02-04.log
jails  jdisable   startup-start  2023-02-04.log
jails  jdisable   startup-stop   2023-02-03.log
jails  jdisable   startup-stop   2023-02-04.log
jails  jds        console        2023-02-03.log
jails  jds        startup-start  2023-02-04.log
jails  jinherit   console        2023-02-03.log
jails  jinherit   console        2023-02-04.log
jails  jinherit   startup-start  2023-02-04.log
jails  jinherit   startup-stop   2023-02-03.log
jails  jinherit   startup-stop   2023-02-04.log
jails  jmultinet  console        2023-02-03.log
jails  jmultinet  startup-stop   2023-02-03.log
jails  jnat       console        2023-02-04.log
jails  jnat       startup-start  2023-02-04.log
jails  jnat       startup-stop   2023-02-04.log
jails  jng        console        2023-02-03.log
jails  jng        startup-start  2023-02-04.log
jails  jnonat     console        2023-02-04.log
jails  jpriv      console        2023-02-04.log
jails  jpub       console        2023-02-04.log
jails  jslaac     console        2023-02-03.log
jails  jslaac     console        2023-02-04.log
jails  jslaac     startup-start  2023-02-04.log
jails  jslaac     startup-stop   2023-02-04.log
jails  jtest      console        2023-02-03.log
jails  jtest      console        2023-02-04.log
jails  jtest      startup-start  2023-02-03.log
jails  jtest      startup-stop   2023-02-03.log
jails  jvirtnet   console        2023-02-03.log
jails  jvnet      console        2023-02-03.log
jails  jvnet      startup-start  2023-02-04.log
jails  jweb       console        2023-02-03.log
jails  jweb       console        2023-02-04.log
jails  jweb       startup-start  2023-02-04.log
jails  jweb       startup-stop   2023-02-03.log
jails  jweb       startup-stop   2023-02-04.log
jails  myjail     console        2023-02-03.log
jails  myjail     console        2023-02-04.log
jails  myjail     startup-start  2023-02-03.log
jails  myjail     startup-start  2023-02-04.log
jails  myjail     startup-stop   2023-02-03.log
jails  myjail     startup-stop   2023-02-04.log
jails  nginx      console        2023-02-04.log
jails  otherjail  console        2023-02-03.log
jails  otherjail  console        2023-02-04.log
jails  otherjail  startup-start  2023-02-03.log
jails  otherjail  startup-start  2023-02-04.log
jails  otherjail  startup-stop   2023-02-03.log
jails  otherjail  startup-stop   2023-02-04.log
jails  php        console        2023-02-04.log
jails  php        console        2023-02-04.log
jails  php        startup-start  2023-02-04.log
jails  php        startup-stop   2023-02-04.log
nat    db         startup-start  2023-02-03.log
nat    db         startup-stop   2023-02-03.log
nat    web        startup-start  2023-02-03.log
nat    web        startup-stop   2023-02-03.log
```

`appjail logs read` uses the `PAGER` environment variable to display the log to read. Set `-R` to display ANSI colors correctly.

```console
# env PAGER="less -R" appjail logs read jails/php/startup-start/2023-02-04.log
[00:00:05] [ debug ] [php] Locking php ...
[00:00:05] [ info  ] [php] Starting php...
[00:00:10] [ debug ] [php] Using `/usr/local/appjail/jails/php/conf/template.conf` as the template.
[00:00:12] [ debug ] [php] Checking for invalid parameters...
[00:00:14] [ debug ] [php] Writing `/usr/local/appjail/jails/php/conf/template.conf` content to `/usr/local/appjail/cache/tmp/.appjail/appjail.wkX25nfm` ...
[00:00:14] [ debug ] [php] Checking for parameters marked as required...
...
```

To remove a log use `appjail logs remove`:

```sh
appjail logs remove jails/jpriv/console/2023-02-04.log
```

Or to remove a bunch of logs in a single command:

```sh
appjail logs remove jails/jpriv
```

Using the `-g` flag we can use shell glob patterns.

```sh
appjail logs remove -g 'jails/otherjail/startup-start/2023-02-0[34].log'
```

!!! warning

    When using ZFS as the backend file system `appjail logs remove` will recursively
    remove all datasets including all references, such as clones. Be careful.

`appjail logs tail` can be used to display the last part of a file.

```sh
appjail logs tail jails/jalias6/startup-stop/2023-02-04.log -f
```

!!! info

    Another useful log is created by the RC script which defaults to `/var/log/appjail.log`.
