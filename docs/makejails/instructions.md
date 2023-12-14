Makejail is a text document that contains all the instructions for building a jail.

Makejail is another layer to abstract many processes to build a jail, configure it, install applications, configure them and much more.

The idea is to provide developers and sysadmins a way to recreate the exact environment that the application needs.

Users of a project containing a Makejail file can use it to install and configure the application with a few commands.

This section describes the commands you can use in a Makejail.

## Specification

### ADD

#### Syntax

```
ADD [--verbose] url [dst]
```

##### --verbose

See`-v, --verbose` in `tar(1)`.

##### url

Tarball URL.

##### dst

The path relative to the jail directory to extract the tarball.

`WORKDIR` affects this command.

#### Description

Download and extract a tarball file.

Useful for `Empty jails`.

#### Examples

```
ADD https://dl-cdn.alpinelinux.org/alpine/v3.17/releases/x86_64/alpine-minirootfs-3.17.0-x86_64.tar.gz
```

### ARG

#### Syntax

```
ARG name[?][=[default_value]]
```

#### Description

Creates arguments to the current stage. See [Initscripts](../initscripts.md) for more details.

If `default_value` is not defined, the argument will be a non-optional argument unless `?=` is used.

This instruction passes its arguments to the `CMD` instruction through environment variables. See [CMD](#cmd) for details.

#### Examples

```
ARG nginx_conf
ARG use_php_fpm=0
```

### CLEAR

#### Syntax

```
CLEAR [entrypoint | user | workdir | stage]
```

##### entrypoint

Resets `ENTRYPOINT` to the default value. See [ENTRYPOINT](#entrypoint_1) for more details.

##### user

Resets `USER` to the default value. See [USER](#user_1) for more details.

##### workdir

Resets `WORKDIR` to the default value. See [WORKDIR](#workdir_1) for more details.

##### stage

Removes all commands written up to this command in the current stage.

#### Description

Resets the value of a command.

#### Examples

```
CLEAR entrypoint
CLEAR user
```

### CMD

#### Syntax

```
CMD --chroot cmd [args ...]
CMD --jaildir cmd [args ...]
CMD [--jexec] [--noclean] [--juser jail_username | --huser host_username] cmd [args ...]
CMD --local cmd [args ...]
CMD --local-jaildir cmd [args ...]
CMD --local-rootdir cmd [args ...]
```

##### --chroot

Use `appjail cmd chroot`.

This parameter uses the jail directory as the chrooted directory.

This does not provide isolation (except at the directory level).

It is not recommended to use thinjails with this parameter.

##### --jaildir

Use `appjail cmd jaildir`.

This parameter uses the directory where the jails are stored.

This does not provide isolation.

##### --jexec (default)

Use `appjail cmd jexec`.

`--noclean` correspond to `appjail cmd jexec -l`

`--juser` correspond to `appjail cmd jexec -U`.

`--huser` correspond to `appjail cmd jexec -u`.

##### --local

Use the current directory. The current directory used by Makejail may be different from the directory where `appjail makejail` was executed (see [INCLUDE](#include) for more details).

This does not provide isolation.

##### --local-jaildir

Use `appjail cmd local -j`.

This parameter uses the jail directory.

This does not provide isolation.

##### --local-rootdir

Use `appjail cmd local -j`.

This parameter uses the root directory of the jail.

This does not provide isolation.

#### Description

This instruction uses the AppJail tokenizer to get a valid posix shell command. It has the responsability to escape harmful characters that can be executed on the host (although this assumption is not valid when using a parameter that executes the command on the host as can be seen in `Syntax`). The idea is that any errors arising from an invalid shell command will only occur on the jail.

The `ARG` instruction can be used in conjunction with this instruction to pass variables. Internally, variables are passed using `env(1)` to the `sh(1)` subprocess that is created when entering the jail to execute the given command.

```
OPTION overwrite
OPTION start

ARG name=DtxdF

CMD echo "Hello, ${name}"
```

The above Makejail will display `Hello, DtxdF` unless you change the variable's value at runtime.

This instruction has the advantage that it can execute virtually any posix shell command.

```
CMD cd /usr/local/etc/opensearch/opensearch-security; for i in $(ls *.sample) ; do cp -p "$i" $(echo $i | sed "s|.sample||g"); done
```

All of the above commands will be executed on the jail, not on the host, even the embedded shell commands.

Remember that this command uses the AppJail tokenizer, so you cannot use an invalid (but accepted by `sh(1)`) shell command. For example, if you run `echo "\"` in a shell script, you will get the error `Syntax error: Unterminated quoted string`, but if you run it in a Makejail you will get `Tokenizer: ERROR [ret:-4, errno:0] <Invalid syntax (WDERRS).>`.

Feel free to use this command, but for more complex things use a shell script. It is recommended to separate complex things into simple things.

#### Examples

##### #1

```
CMD mkdir -p /usr/local/www/darkhttpd
CMD echo "<h1>Hello, world!</h1>" > /usr/local/www/darkhttpd/index.html
```

##### #2

```
CMD --local-jaildir sysrc -f etc/rc.conf clear_tmp_X="NO"
```

### COPY

#### Syntax

```
COPY [--glob | --glob-left | --glob-right] [--verbose] [--jail jail] src [dst]
```

##### --glob

Use the glob expression in the left and right corners of `src`.

##### --glob-left

Use the glob expression in the left corner of `src`.

##### --glob-right

Use the glob expression in the right corner of `src`.

##### --verbose

See `-v` in `cp(1)`.

##### --jail

Copy the file relative to a directory in a jail.

##### src

File to be copied.

If a relative path is used, `appjail makejail` affects this. See [INCLUDE](#include) for more details.

##### dst

The path relative to the jail that is used as the file destination.

`WORKDIR` affects this command.

#### Description

Copy a file from the host to the jail.

#### Examples

##### #1

```
COPY --verbose ${index} /usr/local/www/nginx/index.html
```

##### #2

```
# Will copy /usr/local/lib/{libtag.so,libtag.so.1,libtag.so.1.19.0}
COPY --verbose --jail "${gonic_builder}" --glob-right /usr/local/lib/libtag.so /usr/local/lib
```

### DESTROY

#### Syntax

```
DESTROY [--force] [--with-all-dependents] jail
```

##### --force

See `-f` in `appjail jail destroy`.

##### --with-all-dependents

See `-R` in `appjail jail destroy`.

#### Description

Stops and destroys a jail.

#### Examples

```
DESTROY builder
```

### DEVICE

#### Syntax

```
DEVICE rulespec
```

#### Description

Add a DEVFS rule.

#### Examples

```
DEVICE include 1
DEVICE include 2
DEVICE include 3
DEVICE path fuse unhide
DEVICE path zfs unhide
DEVICE path 'dsp*' unhide
DEVICE path 'mixer*' unhide
DEVICE path bpf unhide
```

### ENTRYPOINT

#### Syntax

```
ENTRYPOINT entrypoint
```

##### entrypoint

The program and, optionally, its arguments.

#### Description

Use `RUN` as arguments to `ENTRYPOINT`.

#### Examples

```
ENTRYPOINT python3.10
RUN script.py
```

### ENV

#### Syntax

```
ENV name[=value]
```

#### Description

Environment variables to be used by the `RUN` command.

We can pass environment variables from the command line using the `-V` parameter supported by `appjail apply`, `appjail makejail`, `appjail start`, `appjail stop` and `appjail run`.

#### Examples

```
ENV TOKEN=bba06278ca32777dc3724d42fe6fd3d9
```

### EXEC

#### Syntax

```
EXEC [--continue-with-errors] [--verbose] [[--after-include include_file] ...] [[--arg parameter[=value]] ...] [[--before-include include_file] ...] [[--build-arg arg] ...] [[--option option] ...] --file makejail --name jail
```

##### --continue-with-errors

See `-e` in `appjail makejail`.

##### --verbose

See `-v` in `appjail makejail`.

##### --after-include

See `-a` in `appjail makejail`.

Global instructions can be used using the global name created by `GLOBAL`. See [GLOBAL](#global) for details.

##### --arg

Sets the value of a parameter to the Makejail to be executed.

##### --before-include

See `-B` in `appjail makejail`.

Global instructions can be used using the global name created by `GLOBAL`. See [GLOBAL](#global) for details.

##### --build-arg

See `-b` in `appjail makejail`.

##### --option

See `-o` in `appjail makejail`.

##### --file

See `-f` in `appjail makejail`.

Global instructions can be used using the global name created by `GLOBAL`. See [GLOBAL](#global) for details.

##### --name

See `-j` in `appjail makejail`.

#### Description

Execute a Makejail.

#### Examples

##### #1

```
EXEC --file gh+AppJail-makejails/hello --name hello
```

##### #2

```
EXEC --file build.makejail --name builder --arg "cflags=-O2 -pipe" --arg ldflags=-lm
```

##### #3

```
EXEC --before-include network.makejail \
     --file other.makejail \
     --name goappb \
     --arg network=development
```

### FROM (build)

#### Syntax

```
FROM [--ajspec ajspec_name] [--entrypoint entrypoint|none] [--platform platform] image[:tag]
```

##### --ajspec

See `-N` in `appjail image import`.

##### --entrypoint

See `appjail image import`.

When no entry point is set, `IMAGE_ENTRYPOINT` (default: `gh+AppJail-makejails`) is concatenated with `image`.

`none` is special because it indicates that the image should not be downloaded, so it is assumed that it is already installed.

##### --platform

See `-a` in `appjail image import` and `appjail image jail`.

##### image

Image name.

##### tag

Tag to be used. If not specified, `IMAGE_TAG` (default: `latest`) is used instead.

#### Description

Use an image as the jail.

#### Examples

```
FROM --entrypoint gh+AppJail-makejails/nginx nginx:13.2
# Equivalent:
FROM nginx:13.2
```

### GLOBAL

#### Syntax

```
GLOBAL :name: [instruction [args ...]]
```

##### name

Global name.

##### instruction

Makejail instruction and, if required, with its arguments.

#### Description

Create a Makejail as a temporary file that can be used by `EXEC`. The intention is to deploy multiple jails using a single Makejail.

Note that since the Makejail is a temporary file, any reference to a file is relative to that directory since `INCLUDE` works this way (see [INCLUDE](#include) for details).

```
# Correct
GLOBAL :darkhttpd: INCLUDE gh+AppJail-makejails/darkhttpd
GLOBAL :darkhttpd: COPY --verbose "${APPJAIL_PWD}/usr/" usr

# Wrong
GLOBAL :darkhttpd: INCLUDE gh+AppJail-makejails/darkhttpd
GLOBAL :darkhttpd: COPY --verbose usr
```

#### Examples

```
# Local options.
OPTION start
OPTION overwrite=force

# Global options.
GLOBAL :network: OPTION virtualnet=:${APPJAIL_JAILNAME} default
GLOBAL :network: OPTION nat

# Web servers.
RAW for jail in nginx darkhttpd; do
    EXEC --after-include :network: \
         --file gh+AppJail-makejails/${jail} \
         --name ${jail}
RAW done

# Command executed by this jail.
CMD echo "Done."
```

### INCLUDE

#### Syntax

```
INCLUDE [method+]path [args ...]
```

##### method

The method that `appjail makejail` will use to get the Makejail file.

##### path

Path to the Makejail file, but this varies depending on the method used.

##### args

Optional arguments for the method used.

#### Description

Includes a Makejail file.

`INCLUDE` removes empty lines, comments and leading spaces.

`INCLUDE` is the first command executed in Makejail when using `appjail makejail -f`.

`INCLUDE` changes the current directory to the directory where Makejail resides if they differ. This allows Makejail to access files relative to its directory.

`INCLUDE` also restores the current stage after reading the current Makejail if the previous stage differs.

After the above processes, `INCLUDE` will include all Makejails in a single Makejail.

The following Makejails ilustrate the above description:

**a.makejail**
```
OPTION start
OPTION overwrite

INCLUDE b.makejail

CMD echo "I'm a in the build stage."
```

**b.makejail**:
```
STAGE cmd

CMD echo "I'm b in the cmd stage."
```

The previous Makejails will be a single Makejail:

```
OPTION start
OPTION overwrite
STAGE cmd
CMD echo "I'm b in the cmd stage."
STAGE build
CMD echo "I'm a in the build stage."
```

To illustrate how `INCLUDE` changes the current directory, the following examples are useful:

**A/Makejail**:
```
OPTION start
OPTION overwrite

CMD echo "I'm A in the build stage."

INCLUDE ../B/Makejail

CMD echo "I'm A in the build stage again."
```

**B/Makejail**:
```
STAGE cmd

CMD echo "I'm B in the cmd stage."

INCLUDE ../C/Makejail
```

**C/Makejail**:
```
STAGE build

CMD echo "I'm C in the build stage."
CMD mkdir -p /usr/local/etc
COPY config.conf /usr/local/etc
CMD cat /usr/local/etc/config.conf

STAGE start
CMD echo "I'm C in the start stage."
```

After including all Makejails in a single Makejail:

```
RAW cd -- "/tmp/n/A" # Makejail: /tmp/n/A/Makejail
OPTION start
OPTION overwrite
CMD echo "I'm A in the build stage."
RAW cd -- "/tmp/n/B" # Makejail: /tmp/n/B/Makejail
STAGE cmd
CMD echo "I'm B in the cmd stage."
STAGE build
RAW cd -- "/tmp/n/C" # Makejail: /tmp/n/C/Makejail
CMD echo "I'm C in the build stage."
CMD mkdir -p /usr/local/etc
COPY config.conf /usr/local/etc
CMD cat /usr/local/etc/config.conf
STAGE start
CMD echo "I'm C in the stage stage."
STAGE cmd
STAGE build
RAW cd -- "/tmp/n/A" # Makejail: /tmp/n/A/Makejail
CMD echo "I'm A in the build stage again."
```

Some `STAGE` commands seem to be uncessary when changing a stage after another stage. The following example illustrates why this is necessary:

**A/Makejail**:
```
OPTION overwrite
OPTION start

CMD echo "I'm A before include B."

INCLUDE ../B/Makejail

CMD echo "I'm A after include B."
```

**B/Makejail**:
```
STAGE start

CMD echo "I'm B in the start stage."
```

The resulting Makejail will be as follows:

```
RAW cd -- "/tmp/c/A" # Makejail: /tmp/c/A/Makejail
OPTION overwrite
OPTION start
CMD echo "I'm A before include B."
RAW cd -- "/tmp/c/B" # Makejail: /tmp/c/B/Makejail
STAGE start
CMD echo "I'm B in the start stage."
STAGE build
RAW cd -- "/tmp/c/A" # Makejail: /tmp/c/A/Makejail
CMD echo "I'm A after include B."
```

In the above example, stage restoration is very important in order not to execute a command in a different stage than the one we intend.

`INCLUDE` can obtain the Makejail file using different methods as mentioned below.

##### file - (syntax: file+makejail_file)

Loads the Makejail from the local file system.

This is the default method.

If the file name contains the `+` sign, you must explicitly use the method.

##### cmd - (syntax: cmd+command [args ...])

Execute a command and use its output (stdout) as the Makejail file.

##### git - (syntax: git+url [--baseurl url] [--file makejail_filename] [--global | --local [--cachedir directory] | --tmp])

Clone a `git(1)` repository in the global cache directory (`GLOBAL_GIT_CACHEDIR`) or in the local cache directory specified with `--cachedir` (default: `.makejail_local`) to get the Makejail named `makejail_filename` (default: `Makejail`). Using the `--tmp` parameter uses a temporary directory as the cache directory, so the `git(1)` repository will be cloned each time.

The `--basedir` parameter is used as a URL prefix and is intended for other git-like methods, such as those mentioned in the following sections.

`devel/git` must be installed before using this method.

##### fetch - (syntax: url)

Use `MAKEJAIL_FETCH_CMD` to make an HTTP or FTP request to get the Makejail file.

##### gh, github - (syntax: USERNAME/REPONAME)

Wrapper of the `git` method setting `--baseurl` to `https://github.com/`.

##### gh-ssh, github-ssh - (syntax: USERNAME/REPONAME)

Wrapper of the `git` method setting `--baseurl` to `git@github.com:`.

##### gl, gitlab - (syntax: USERNAME/REPONAME)

Wrapper of the `git` method setting `--baseurl` to `https://gitlab.com/`.

##### gl-ssh, gitlab-ssh - (syntax: USERNAME/REPONAME)

Wrapper of the `git` method setting `--baseurl` to `git@gitlab.com:`.

#### Examples

##### #1

```
# Identical.
INCLUDE /tmp/Makejail
INCLUDE file+/tmp/Makejail
```

##### #2

```
# Since there is a plus sign in the file name, we must use
# the method explicitly.
INCLUDE file+/home/op/tmp/xeyes+debian-bullseye.makejail
```

##### #3

```
INCLUDE gh+AppJail-makejails/python
```

##### #4

```
# Identical, but it is recommend to use fetch to honor
# MAKEJAIL_FETCH_CMD.
INCLUDE fetch+https://example.org/nginx-makejail
INCLUDE cmd+fetch -o - https://example.org/nginx-makejail
```

### MOUNT

#### Syntax

```
MOUNT --nopersist device mountpoint [type] [options] [dump] [pass]
MOUNT [--nomount] [--nro [auto | nro]] device mountpoint [type] [options] [dump] [dump]
```

##### --nopersist

By default, `MOUNT` uses `appjail fstab`, this option uses `mount(8)` instead.

##### --nomount

By default, `appjail fstab` fields are compiled and mounted, this option disables it.

If you use `MOUNT` serveral times, it is recommended to use this option except the last time it is called at the same stage.

##### device

Device to be mounted.

##### mountpoint

Mountpoint relative to the jail directory.

##### type

File system type.

The default is `nullfs`.

##### options

Options for `mount(8)`.

The default is `rw`.

##### dump

This field is used for these file systems by the `dump(8)` command to determine which file systems need to be dumped. See `fstab(5)` for more details.

The default is `0`.

##### pass

This field is used by the `fsck(8)` and `quotacheck(8)` programs to determine the order in which file system and quota checks are done at reboot time. See `fstab(5)` for more details.

The default is `0`.

#### Description

Mount a device inside the jail.

The AppJail tokenizer allows you to use quoted strings to use spaces.

#### Examples

##### #1

```
MOUNT /usr/ports /usr/ports
```

##### #2

```
MOUNT "/tmp/with spaces" /tmp/n
```

##### #3

```
MOUNT --nomount /usr/ports /usr/local/www
MOUNT --nomount /usr/local/www /usr/local/www
MOUNT /tmp /tmp
```

### PKG

#### Syntax

```
PKG [[--chroot | --jexec [--jail] | --local]] package ...
PKG [[--chroot | --jexec [--jail] | --local]] --remove package ...
PKG [[--chroot | --jexec [--jail] | --local]] --autoremove
PKG [[--chroot | --jexec [--jail] | --local]] --clean
PKG [[--chroot | --jexec [--jail] | --local]] --update
PKG [[--chroot | --jexec [--jail] | --local]] --upgrade
```

##### --chroot

Use `pkg(8)` in the jail directory as the new root directory.

This option can be used for thinjails and thickjails. For thinjails it is necessary that the jail is started.

##### --jexec

`pkg(8)` will execute in the given jail.

`--jail` is to bootstrap `pkg(8)` inside the jail before use.

##### --local

Run the host's package manager.

##### --remove

Remove one or more packages instead of installing them.

##### --autoremove

Remove orphan or unused packages.

##### --clean

Clean the local cache of fetched remote packages.

##### --update

Update the list of packages.

##### --upgrade

Perform upgrades of package software distributions.

##### package

Package name.

Can be used several times.

#### Description

Use `pkg-install(8)` to install a package.

#### Examples

```
PKG nginx
```

### RAW

#### Syntax

```
RAW [code]
```

#### Description

Write `sh(1)` code. Useful for conditionals, loops and anything you can to do with `sh(1)`.

#### Examples

##### #1

```
RAW variable="value"
```

##### #2

```
RAW if [ "${use_php}" != 0 ]; then
    PKG php
RAW fi
```

### REPLACE

#### Syntax

```
REPLACE file old [new] [output]
```

##### file

File containing the keywords to be replaced.

##### old

The keyword to be replaced.

##### new

Replaces the given keyword using this value. If not value is given, an empty value will be used.

##### output

Instead of replacing `file`, use `output` as the new file with the replaced keywords.

#### Description

Replace a keyword (e.g.: `%{VARIABLE}`) for a specific value in the given file.

To use the literal keyword use the `%` twice. For example, the `%%{VARIABLE}` keyword will convert to `%{VARIABLE}`.

#### Examples

```
REPLACE /usr/local/www/wordpress/wp-config-appjail.php DB_NAME wordpress /usr/local/www/wordpress/wp-config.php
```

### RUN

#### Syntax

```
RUN [--maintain-env] [--noclean] [--juser jail_username | --huser host_username] [cmd [args ...]]
```

!!! note

    The arguments have the same meaning as `CMD --jexec`.

##### --maintain-env

Leave the environment unchanged instead of simulating a full login.

#### Description

Execute a program.

Unlike `CMD`, `RUN` does not execute shell code. `RUN` only passes its arguments to `ENTRYPOINT` with a literal meaning.

`RUN` does not use shell variables like arguments (see [ARG](#arg)) or variables (see [VAR](#var)). `RUN` uses environment variables created by the `ENV` command.

`RUN` will be executed as the user specified by the `USER` command.

`RUN` will execute the command in the directory specified by the `WORKDIR` command.

`RUN` cannot run programs in interactive mode like `python`. Use `CMD` for this.

#### Examples

##### #1

```
# Prints the  `Hello, world! > /tmp/hello.txt` message instead of writing it to the `/tmp/hello.txt` message.
RUN echo "Hello, world!" > /tmp/hello.txt
```

##### #2

```
RUN python3.9 app.py
```

### SERVICE

#### Syntax

```
SERVICE service_args
```

#### Description

Manipulate services in the jail.

#### Examples

```
SERVICE nginx nginx oneenable
SERVICE nginx nginx start
```

### SET

#### Syntax

```
SET [--mark] [--column column] [--row row] parameter[=value]
```

##### --mark

Mark a parameter as required. See [Templates](../templates.md) for more details.

##### --column, --row

The column and the row to edit. See [Templates](../templates.md) for more details.

#### Description

Use `appjail-config` to edit the current template.

#### Examples

```
# A basic linux template is as follows:
SET exec.start=/bin/true
SET exec.stop=/bin/true
SET persist
```

### STAGE

#### Syntax

```
STAGE stage
```

#### Description

Change the current stage.

The default stage is `build`.

#### Examples

##### #1

```
STAGE start
```

##### #2

```
STAGE custom:python
```

### SYSRC

#### Syntax

```
SYSRC [--jail | --local] name[[+|-]=value] ...
```

##### --jail

Use `jexec(8)` to edit the rc file within the jail.

This is the default.

##### --local

Use the jail directory as the new root directory to edit the rc file.

Use this parameter for thickjails, it will probably not work in a thinjail.

#### Description

Use `sysrc(8)` to edit rc files of the jail.

#### Examples

```
SYSRC nginx nginx_enable="YES"
```

### UMOUNT

#### Syntax

```
UMOUNT mountpoint
```

#### Description

Unmount a mounted file system.

This command does not use the AppJail tokenizer, so there is no need to use quoted strings.

#### Examples

```
UMOUNT /usr/local/www/darkhttpd
```

### USER

#### Syntax

```
USER user
```

#### Description

The user that the `RUN` command will use to execute a command as the given user.

#### Examples

```
USER xclock
```

### VAR

#### Syntax

```
VAR [--make-arg-env] [--noexpand] name[=default_value]
```

##### --make-arg-env

Create an environment variable to be passed to `CMD`. See [CMD](#cmd) for details.

##### --noexpand

When this option is used, the `$` sign is escaped. Useful for `build arguments`.

#### Description

Create or set a variable.

#### Examples

```
VAR wwwdir=/usr/local/www
```

### WORKDIR

#### Syntax

```
WORKDIR workdir
```

#### Description

Creates a new directory and uses it as a working directory for some commands such as `ADD`, `COPY` and `RUN`.

#### Examples

```
WORKDIR /app
COPY app.py
RUN python3.9 app.py
```

### LOGIN (build)

#### Syntax

```
LOGIN [--user username]
```

##### --user

The username to try to log in.

The default user is `root`.

#### Description

Use `appjail login` to log into the jail.

#### Examples

```
LOGIN
```

### OPTION (build)

#### Syntax

```
OPTION option
```

#### Description

Options to be used in the `appjail quick` command.

#### Examples

```
OPTION virtualnet=web:nginx default
OPTION expose=80
OPTION limits=vmemoryuse:deny=512m
OPTION nat
OPTION overwrite
OPTION start
```

### RESTART (build)

#### Syntax

```
RESTART
```

#### Description

Use `appjail restart` to restart the jail.

#### Examples

```
RESTART
```

### START (build)

#### Syntax

```
START
```

#### Description

Use `appjail start` to start the jail.

#### Examples

```
START
```

### STOP (build)

#### Syntax

```
STOP
```

#### Description

Use `appjail stop` to stop the jail.

#### Examples

```
STOP
```

### VOLUME

#### Syntax

```
VOLUME [--group gid] [--mountpoint mountpoint] [--owner uid] [--perm mode] [--type fs_type] volume
```

##### --group

Changes the group ID of the specified volume.

##### --mountpoint

Path within the jail to mount the specified volume. Default is `{VOLUMESDIR}/{volume_name}`.

##### --owner

Changes the user ID of the specified volume.

##### --perm

Changes the file mode of the specified volume.

##### --type

File system type. Valid are `nullfs` and `<pseudofs>`.

#### Examples

```
VOLUME db
```
