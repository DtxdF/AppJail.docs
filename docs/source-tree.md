If you need more control or just want to take advantage of your CPU, you can compile the FreeBSD source tree in at least three ways using AppJail.

### fetch src

This is a method of the `fetch` command and its use is very simple.

```console
# appjail fetch src
[00:00:02] [ info  ] Build log will be releases/default/build/2023-10-12_06h18m40s.log
[00:00:02] [ info  ] Starting installworld with 8 jobs ...
[00:03:57] [ info  ] installworld finished!
[00:03:57] [ info  ] Starting distrib-dirs ...
[00:04:02] [ info  ] distrib-dirs finished!
[00:04:02] [ info  ] Starting distribution ...
[00:04:36] [ info  ] distribution finished!
[00:04:36] [ info  ] Starting delete-old delete-old-libs ...
[00:05:28] [ info  ] delete-old delete-old-libs finished!
```

The above command assumes a few things. First, that you already have a source tree on your system, and second, that you ran `buildworld` previously. Since both assumptions may not make sense in your case, the first thing to do is to obtain the source tree.

```sh
git clone --branch releng/13.2 --depth 1 https://github.com/freebsd/freebsd-src.git /usr/src
```

You did a shallow clone of a source tree that points to the `releng/13.2` branch.

Let's build the world.

```console
# appjail fetch src -b
[00:00:01] [ info  ] Build log will be releases/default/build/2023-10-12_06h33m06s.log
[00:00:01] [ info  ] Starting buildworld with 8 jobs ...
[00:14:00] [ info  ] buildworld finished!
[00:14:00] [ info  ] Starting installworld with 8 jobs ...
[00:18:14] [ info  ] installworld finished!
[00:18:14] [ info  ] Starting distrib-dirs ...
[00:18:20] [ info  ] distrib-dirs finished!
[00:18:20] [ info  ] Starting distribution ...
[00:18:55] [ info  ] distribution finished!
[00:18:55] [ info  ] Starting delete-old delete-old-libs ...
[00:19:48] [ info  ] delete-old delete-old-libs finished!
```

As you can see, just add `-b` and AppJail will do all the work for you.

If you want to read the build in progress:

```sh
appjail logs tail releases/default/build/2023-10-12_06h33m06s.log -f
```

Of course, this compilation has been completed, so you'll probably just want to get it all.

```sh
appjail logs read releases/default/build/2023-10-12_06h33m06s.log
```

By default only the world is compiled, but you can indicate AppJail that you want to compile the kernel (not necessary in almost all cases, except when you have to).

```console
# appjail fetch src -bk
[00:00:01] [ info  ] Build log will be releases/default/build/2023-10-12_06h53m47s.log
[00:00:01] [ info  ] Starting buildworld with 8 jobs ...
[00:13:38] [ info  ] buildworld finished!
[00:13:38] [ info  ] Starting buildkernel with 8 jobs ...
[00:14:28] [ info  ] buildkernel finished!
[00:14:28] [ info  ] Starting installworld with 8 jobs ...
[00:18:29] [ info  ] installworld finished!
[00:18:29] [ info  ] Starting distrib-dirs ...
[00:18:35] [ info  ] distrib-dirs finished!
[00:18:35] [ info  ] Starting distribution ...
[00:19:09] [ info  ] distribution finished!
[00:19:09] [ info  ] Starting installkernel with 8 jobs ...
[00:19:40] [ info  ] installkernel finished!
[00:19:40] [ info  ] Starting delete-old delete-old-libs ...
[00:20:35] [ info  ] delete-old delete-old-libs finished!
```

`-k` is sufficient, AppJail just fires.

To indicate AppJail to cross-compile the world to another architecture, say, `i386`, just add `-a i386`.

```sh
appjail fetch src -a i386
```

This will set `TARGET` to `i386` and `TARGET_ARCH` will be left empty, so the Makefile in the source tree will use a reasonable value. To specify `TARGET_ARCH` explicitly just add `-a target/target_arch`, for example, `-a arm/armv7`.

Now we have a release, we can create a jail just like in a standard way.

```sh
appjail quick jtest \
    overwrite=force \
    start \
    osarch=i386 \
    osversion=13.2-RELEASE
```

#### Naming convention

As you can see above, the release name is used by default, which is not recommended when using `appjail-fetch(1)` `src`.

```console
# appjail fetch list
ARCH   VERSION       NAME
...
amd64  13.2-RELEASE  default
...
```

As you can see, there is not enough information for this release. Of course, the architecture, the version and the name, but when using `appjail-fetch(1)` `src` AppJail will set `ARCH` to `TARGET` instead of `TARGET_ARCH`. As you can see below there is an ambiguity.

```console
# make -C /usr/src targets
Supported TARGET/TARGET_ARCH pairs for world and kernel targets
    amd64/amd64
    arm/armv6
    arm/armv7
    arm64/aarch64
    i386/i386
    mips/mips
    mips/mips64
    powerpc/powerpc
    powerpc/powerpc64
    riscv/riscv64
    riscv/riscv64sf
```

It is fine for `amd64` and `i386`, but not for `arm` or other architecture. What we can do is set the release name to a meaningful value.

```console
# appjail fetch src -a arm armv7
...
# appjail fetch list
ARCH   VERSION       NAME
...
arm    13.2-RELEASE  armv7
...
```

There is a subtle difference when creating a jail: we need to add `release=<release name>`.

```sh
appjail quick jtest \
    overwrite=force \
    start \
    osarch=arm \
    osversion=13.2-RELEASE \
    release=armv7
```

#### Update

AppJail can update both jails and releases easily. Of course, updating an installed jail or a release coming from a source tree is different from doing it using the binary form (aka: `freebsd-update(8)`).

##### jail

The requirement to update a jail is that it must be a thickjail, but to update using the source tree, i.e. `appjail-update(1)` `jail`, will make some assumptions. If the jail has a release directory that has the dummy file `.from_src`, it means that this jail was installed using a source tree, so `appjail-update(1)` `jail` should proceed to update the jail using the source instead of the binary form. This, of course, creates a link between the jail and the release, which we must take into account when we are going to destroy the release, update the jail or export it to another system.

If we export the jail to another system we must export/import the release to that system unless we don't need to use the source tree of that system anymore. As a tip, you can export the jail and export the release with only the dummy files to have less files to send, ignoring the content of the `release` directory, but of course, you will need to install the world and the kernel (if you want to).

Suppose we created a jail named `jtest`, which is a thickjail (thinjails can be used, of course, but you cannot update a thinjail, so it is useless for the following example).

```sh
appjail quick jtest \
    overwrite=force \
    start \
    osarch=amd64 \
    osversion=13.2-RELEASE \
    type=thick
```

Run the extract mode. This is only needed once.

```console
# appjail etcupdate jail -m extract jtest
[00:00:00] [ info  ] [jtest] etcupdate(8) log will be jails/jtest/etcupdate/2023-10-12_09h54m35s.log
```

Before running `installworld` we need to run `etcupdate -p`:

```console
# appjail etcupdate jail jtest -p
[00:00:00] [ info  ] [jtest] etcupdate(8) log will be jails/jtest/etcupdate/2023-10-12_10h01m40s.log
```

Now update the jail (and also build the world and, if any, the kernel):

```console
# appjail update jail -b jtest
[00:00:01] [ info  ] [jtest] Build log will be jails/jtest/build/2023-10-12_10h03m16s.log
[00:00:01] [ info  ] [jtest] Starting buildworld with 8 jobs ...
[00:13:04] [ info  ] [jtest] buildworld finished!
[00:13:04] [ info  ] [jtest] Starting buildkernel with 8 jobs ...
[00:13:53] [ info  ] [jtest] buildkernel finished!
[00:13:53] [ info  ] [jtest] Starting installworld with 8 jobs ...
[00:18:01] [ info  ] [jtest] installworld finished!
[00:18:01] [ info  ] [jtest] Starting installkernel with 8 jobs ...
[00:18:33] [ info  ] [jtest] installkernel finished!
[00:18:33] [ info  ] [jtest] Done.
```

As you can see, AppJail not only build and install the world but also build and install the kernel. AppJail knows this depending on whether you successfully install the kernel when executing `appjail-fetch(1)` `src`. You can avoid running the `buildkernel` and `installkernel` target by using the `-K` parameter in `appjail-update(1)` `jail`.

AppJail needs to know some details when running the required targets, such as `TARGET`, `TARGET_ARCH` (if any), `KERNCONF`, and the source tree. AppJail knows these using dummy files of the release directory of that jail.

As you can see, there are some missing targets that are not executed compared to when we run `appjail-fetch(1)` `src`, this is because you first need to run `etcupdate -B` and `delete-old` and `delete-old-libs` targets for yourself when you consider necessary.

```console
# appjail etcupdate jail jtest -B
[00:00:00] [ info  ] [jtest] etcupdate(8) log will be jails/jtest/etcupdate/2023-10-12_10h24m03s.log
# appjail checkOld jail jtest
...
# appjail deleteOld jail jtest
...
```

##### release

Although you can perfectly run `appjail-update(1)` `release`, this will not work as expected.

```console
# appjail update release
[00:00:00] [ error ] [default] This is a release installed from a source tree, so you will have to run `appjail fetch src` by yourself.
```

As the command guesses, we need to use `appjail-fetch(1)` `src`. This is because `appjail-update(1)` `release` would not be able to guess a perfect workflow for all users, so it is preferable to leave the path clean for the user.

As you can see in the previous sections, all the targets that are needed to create a clean release are executed, but this implies that if we call `appjail-fetch(1)` `src` again, some files will be overwritten. This is fine if you don't have any problems. For example, if you have a thinjail, and you separate the data that must persist (mounted from the host to inside the jail) from the data that is ephemeral, you can recreate the jail and everything will work as expected, plus the new data from the recent update. If you have a thickjail, no problem, remember that the release and a thickjail does not share data in a similar way to thinjails. But if you really want to update a release like jails (see previous section), you just have to indicate `appjail-fetch(1)` `src` not to execute some targets, namely `distrib-dirs`, `distribution`, `delete-old` and `delete-old-libs`. Before updating, you should have run `etcupdate extract` (this is only needed once).

```console
# appjail etcupdate release -m extract
[00:00:00] [ info  ] [default] etcupdate(8) log will be releases/default/etcupdate/2023-10-12_12h59m53s.log
# appjail etcupdate release - -p
[00:00:00] [ info  ] [default] etcupdate(8) log will be releases/default/etcupdate/2023-10-12_13h12m35s.log
# appjail fetch src -bDNR
[00:00:01] [ info  ] Build log will be releases/default/build/2023-10-12_13h15m19s.log
[00:00:01] [ info  ] Starting buildworld with 8 jobs ...
[00:11:43] [ info  ] buildworld finished!
[00:11:43] [ info  ] Starting installworld with 8 jobs ...
[00:15:45] [ info  ] installworld finished!
```

Once the previous work has been completed, it is sufficient to execute the rest.

```console
# appjail etcupdate release - -B
...
# appjail checkOld release
...
# appjail deleteOld release
...
```

### Empty

The other two ways to compile the source tree in AppJail is manually, through a single jail or a release that can be used by other jails. There is not much difference in how we use it to compile the source tree since they are empty directories, it is our responsibility to set `DESTDIR` to the appropriate directory.

#### jail

There is not much magic in the following command, it simply creates an empty directory.

```console
# appjail quick jtest empty overwrite=force noresolv_conf notzdata
...
# appjail cmd local jtest pwd
/usr/local/appjail/jails/jtest/jail
```

#### release

When we use `appjail-fetch(1)` `empty`, AppJail does not know what release and architectures we are going to use, although we can specify them using `-a` and `-v`, but for this example we do not specify them.

```console
# appjail fetch empty
/usr/local/appjail/releases/any/any/default
```

If the directory does not exist, it is created and displayed in the output. If `ls(1)` in it we see some interesting files.

```console
# ls -A /usr/local/appjail/releases/any/any/default
.done   .empty  release
```

There are three files. `.done` is to indicate to `fetch empty` that this release has been successfully created, but it is irrelevant for this case. `.empty` is a hint for other commands like `appjail-update(1)` `release` and `appjail-upgrade(1)` `release` not to update/upgrade this release. And finally and most importantly, the `release` directory is empty and is the one we use to put the release files.

#### Profit!

```sh
# release
D=/usr/local/appjail/releases/any/any/default/release
# or jail
D=/usr/local/appjail/jails/jtest/jail

make -C /usr/src installworld -j$(nproc) DESTDIR=$D DB_FROM_SRC=1
make -C /usr/src distrib-dirs -j$(nproc) DESTDIR=$D DB_FROM_SRC=1
make -C /usr/src distribution -j$(nproc) DESTDIR=$D DB_FROM_SRC=1
```

In the case of a jail, when you successfully compile the source tree into it, just start it and play with it. When it comes to a release, just create a jail and play with it. That's it.

#### Update / Upgrade

If an attempt is made to update or upgrade an empty jail or an empty release, the following will occur:

```console
# appjail update release -a any -v any default
[00:00:00] [ error ] [default] This is an empty release, so you will have to update it manually.
# appjail upgrade release -u -n 13.2-RELEASE -a any -v any default
[00:00:00] [ error ] [default] This is an empty release, so you will have to upgrade it manually.
# appjail update jail jtest
[00:00:00] [ error ] [jtest] jtest is not a thickjail.
```

!!! note

    As you can guess, the empty release follows the same approach as jails that come from a source tree, since they have a dummy file indicating that they are an empty release, jails that use them depend on said release directory and its files.
