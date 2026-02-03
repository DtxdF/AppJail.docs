## Instructions

### ADD

```
ADD [--verbose] url [destination]
```

Download and extract a tarball file from <ins>url</ins> to the jail directory or <ins>destination</ins>, a path relative to the jail directory, using the program specified by the `MAKEJAIL_ADD_FETCH_CMD` parameter. Use `--verbose` to produce a verbose output when extracting the tarball using `tar(1)`.

`WORKDIR` can affect this instruction.

### ARG

```
ARG name[[?]=[default-value]]
```

Create arguments as specified by `appjail-initscript(5)`. If no default value is specified, the arguments become non-optional unless `?` is used. When `?` is used, the user can set a parameter to an empty value.

You don't need the argument value enclosed in double quotes even when it has spaces.

See `CMD` for how arguments are passed to the process.

### CLEAR

```
CLEAR [entrypoint|user|workdir|stage]
```

Clear the value previously set by one of the following instructions: `ENTRYPOINT`, `USER`, `WORKDIR` or `STAGE`.

In the case of the `STAGE` instruction, all commands written up to this instruction and in the current stage are removed.

### CMD

```
CMD --chroot command [args ...]
CMD --jaildir command [args ...]
CMD [--jexec] [--noclean] [[--env name[=value]] ...] [--juser
    username|--huser username] [--workdir working-directory] command
    [args ...]
CMD --local command [args ...]
CMD --local-jaildir command [args ...]
CMD --local-rootdir command [args ...]
```

This instruction uses the AppJail tokenizer best described in `appjail-template(5)` to execute a string with `sh(1)` instructions. This instruction keeps the ", ', and \ characters in the string to better emulate the behavior of `sh(1)`.

Internally, `ARG` and `VAR` (it doesn't do this by default, but does it with one of its parameters) can create variables that are passed via environment variables to the `sh(1)` process.

```
OPTION overwrite
OPTION start

ARG name=DtxdF

CMD echo "Hello, ${name}"
```

This instruction has the advantage that it can execute virtually any shell command.

```
CMD cd /usr/local/etc/opensearch/opensearch-security; for i in $(ls *.sample) ; do cp -p "$i" $(echo $i | sed "s|.sample||g"); done
```

All of the above commands will be executed on the jail, not on the host, even the embedded shell commands.

Remember that this command uses the AppJail tokenizer, so you cannot use an invalid (but accepted by `sh(1)`) shell command. For example, if you run `echo "\"` in a shell script, you will get the error "*Syntax error: Unterminated quoted string*" but if you run it in a Makejail you will get "*Tokenizer: ERROR [ret:-4, errno:0] <Invalid syntax (WDERRS).\>.*"

`--chroot` is equivalent to `appjail-cmd(1)` `chroot`.

`--jaildir` is equivalent to `appjail-cmd(1)` `jaildir`.

`--jexec` `--noclean` is equivalent to `appjail-cmd(1)` `jexec` `-l`.
`--jexec` `--env` is equivalent to `appjail-cmd(1)` `jexec` `-e`.
`--jexec` `--juser` is equivalent to `appjail-cmd(1)` `jexec` `-U`.
`--jexec` `--huser` is equivalent to `appjail-cmd(1)` `jexec` `-u`.
`--jexec` `--workdir` is equivalent to `appjail-cmd(1)` jexec `-w`.

`--local` runs a command from the host but using the local directory which may be different. See `INCLUDE` for more details.

`--local-jaildir` is equivalent to `appjail-cmd(1)` `local` `-j`.

`--local-rootdir` is equivalent to `appjail-cmd(1)` `local` `-r`.

### COPY

```
COPY [--glob|--glob-left|--glob-right] [--verbose] [--jail jail] source
        [destination]
```

Copy a file from the host to the jail or destination, a path relative to the jail directory.

`WORKDIR` can affect this instruction.

Use `--jail` to copy source from another jail.

Using `--glob`, `--glob-left`, `--glob-right` is equivalent to `*source*`, `*source` and `source*`, but you can't use such expressions in `source`.

Increase the verbosity using `--verbose`.

Note that this instruction copies the file or directory as is, that is, metadata such as file mode, uid, gid, etc., are preserved.

### DESTROY

```
DESTROY [--force] [--with-all-dependents] jail
```

Stop and destroy <ins>jail</ins>.

`--force` is equivalent to `appjail-jail(1)` `destroy` `-f`.

`--with-all-dependents` is equivalent to `appjail-jail(1)` `destroy` `-R`.

### DEVICE

```
DEVICE rulespec ...
```

Apply (see `appjail-devfs(1)` `apply`) and `add` (see `appjail-devfs(1)` `set`) a new DEVFS rule.

### ENTRYPOINT

```
ENTRYPOINT program
```

When running a program using `RUN`, the program specified by this instruction is used implicitly, for example:

```
ENTRYPOINT python3.10
RUN script.py
```

### ENV

```
ENV name[=value]
```

Environment variables used by `RUN`. Additional environment variables can be passed using the `-V` parameter supported by `appjail-apply(1)`, `appjail-makejail(1)`, `appjail-start(1)`, `appjail-stop(1)` and `appjail-run(1)`.

### EXEC

```
EXEC [--continue-with-errors] [--verbose] [[--after-include makejail]
        ...] [[--arg parameter[=value]] ...] [[--before-include makejail]
        ...] [[--build-arg parameter[=value]] ...] [[--option option] ...]
        --file makejail --name name
```

Run a Makejail.

`--continue-with-errors` is equivalent to `appjail-makejail(1)` `-e`.

`--verbose` is equivalent to `appjail-makejail(1)` `-v`.

`--after-include` is equivalent to `appjail-makejail(1)` `-a`.

`--arg` is equivalent to passing arguments as you normally do from command-line but without double dashes.

`--before-include` is equivalent to `appjail-makejail(1)` `-B`.

`--option` is equivalent to `appjail-makejail(1)` `-o`.

`--file` is equivalent to `appjail-makejail(1)` `-f`.

`--name` is equivalent to `appjail-makejail(1)` `-j`.

`--file`, `--after-include` and `--before-include` can use a temporary Makejail defined by `GLOBAL`.

### FROM

```
FROM [--ajspec name] [--branch branch] [--entrypoint [entrypoint|none]]
     [--platform platform] image[:tag]
```

Import an image to create a jail.

`--ajspec` is equivalent to `appjail-image(1)` `import` `-N`.

If `--entrypoint` is not specified, this instruction does what `IMAGE_ENTRYPOINT` describes. If set to `none`, it is assumed that the image is currently installed, so this instruction will not attempt to download it. See `appjail-image(1)` `import` for more details.

`--branch` is equivalent to `appjail-image(1)` `import` `-b`.

`--platform` is equivalent to `appjail-image(1)` `import` `-a`.

<ins>image</ins> is equivalent to `appjail-image(1)` `import` `-n`.

<ins>tag</ins> is equivalent to `appjail-image(1)` `import` `-t`. If not defined, the tag specified by the `IMAGE_TAG` parameter is used.

### GLOBAL

```
GLOBAL :name: [instruction [args ...]]
```

Create a temporary Makejail that can be executed by `EXEC`. This instruction is intended for those who want to build, from another jail, an application that generates an executable that is copied by the main Makejail and used by the main jail, although this instruction can be used for much more, for example deploying multiple jails whose services are used by the main jail. However, nothing prevents you from creating another Makejail file and configuring the `EXEC` instruction to use it.

Note that since the Makejail generated by this instruction is a temporary file, any reference to a file is relative to that directory since `INCLUDE` works this way.

```
### Correct
GLOBAL :darkhttpd: INCLUDE gh+AppJail-makejails/darkhttpd
GLOBAL :darkhttpd: COPY --verbose "${APPJAIL_PWD}/usr/" usr
```

```
### Wrong
GLOBAL :darkhttpd: INCLUDE gh+AppJail-makejails/darkhttpd
GLOBAL :darkhttpd: COPY --verbose usr
```

### INCLUDE

This is the first instruction executed, which includes a Makejail file, removes empty lines and comments, changes the current directory to the directory where the included Makejail is located, and restores the previous stage after reading the last included Makejail. After doing all this, internally a single Makejail file will be written with all the instructions from all the other Makejails (except the `INCLUDE` instructions, of course) which is finally executed.

The following Makejails ilustrate the above description:

**a.makejail**:

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

The resulting Makejail will be:

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
RAW cd -- "/tmp/n/A" ### Makejail: /tmp/n/A/Makejail
OPTION start
OPTION overwrite
CMD echo "I'm A in the build stage."
RAW cd -- "/tmp/n/B" ### Makejail: /tmp/n/B/Makejail
STAGE cmd
CMD echo "I'm B in the cmd stage."
STAGE build
RAW cd -- "/tmp/n/C" ### Makejail: /tmp/n/C/Makejail
CMD echo "I'm C in the build stage."
CMD mkdir -p /usr/local/etc
COPY config.conf /usr/local/etc
CMD cat /usr/local/etc/config.conf
STAGE start
CMD echo "I'm C in the stage stage."
STAGE cmd
STAGE build
RAW cd -- "/tmp/n/A" ### Makejail: /tmp/n/A/Makejail
CMD echo "I'm A in the build stage again."
```

Some `STAGE` instructions seem unnecessary, but are relevant in some cases, for example:

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
RAW cd -- "/tmp/c/A" ### Makejail: /tmp/c/A/Makejail
OPTION overwrite
OPTION start
CMD echo "I'm A before include B."
RAW cd -- "/tmp/c/B" ### Makejail: /tmp/c/B/Makejail
STAGE start
CMD echo "I'm B in the start stage."
STAGE build
RAW cd -- "/tmp/c/A" ### Makejail: /tmp/c/A/Makejail
CMD echo "I'm A after include B."
```

The previous example illustrates the importance of restoring the stage so as not to execute instructions at a different stage than intended.

A Makejail can be included in several ways, depending on the <ins>method</ins> used:

**file**+<ins>makejail</ins>

Include a Makejail file from the local file system. This is the default method.

Note that you must set this method explicitly when the pathname has a `+` sign.

**cmd**+<ins>command</ins> [<ins>args</ins> ...]

Use the output of a command as the Makejail file.

**git**+<ins>url</ins> [**--baseurl** <ins>url</ins>] [**--branch** <ins>branch</ins>] [**--file** <ins>makejail</ins>] [**--global**|**--local** [**--cachedir** <ins>directory</ins>|**--tmp**]

Clone a `git(1)` repository.

With `--global`, the `git(1)` repository is cloned to the global cache directory defined by `GLOBAL_GIT_CACHEDIR`, with `--local`, the `git(1)` repository is cloned to the local cache directory defined by the `--cachedir` parameter, and with `--tmp` the `git(1)` repository is cloned as a temporary directory.

After the `git(1)` repository is cloned, the Makejail specified by `--file`, which by default is Makejail, is executed.

`--basedir` is intended for other git-like methods.

By default no branch is specified, but with `--branch` you can specify a specific branch.

This instruction requires that `devel/git` be installed before use.

**fetch**+<ins>url</ins>

Use the program specified by `MAKEJAIL_FETCH_CMD` to download the Makejail file.

**gh**+<ins>username</ins>/<ins>reponame</ins>

**github**+<ins>username</ins>/<ins>reponame</ins>

Wrapper for the `git` method but with `--basedir` set to `https://github.com/`.

**gh-ssh**+<ins>username</ins>/<ins>reponame</ins>

**github-ssh**+<ins>username</ins>/<ins>reponame</ins>

Wrapper for the `git` method but with `--basedir` set to `git@github.com:`.

**gl**+<ins>username</ins>/<ins>reponame</ins>

**gitlab**+<ins>username</ins>/<ins>reponame</ins>

Wrapper for the `git` method but with `--basedir` set to `https://gitlab.com/`.

**gl-ssh**+<ins>username</ins>/<ins>reponame</ins>

**gitlab-ssh**+<ins>username</ins>/<ins>reponame</ins>

Wrapper for the `git` method but with `--basedir` set to `git@gitlab:`.

### LABEL

```
LABEL key[=value]
```

Add a new label to the jail.

### MOUNT

```
MOUNT --nopersist device mountpoint [type]
        [options] [dump] [pass]
MOUNT [--nomount] [--nro [auto|nro]] device mountpoint [type] [options]
        [dump] [pass]
```

Mount file systems inside the jail.

This instruction simulates an `fstab(5)` entry as you can see, but unlike it only <ins>device</ins> and <ins>mountpoint</ins> are required, and the others, <ins>type</ins> (default: **nullfs**), <ins>options</ins> (default: **rw**), <ins>dump</ins> (default: **0**) and <ins>pass</ins> (default: **0**) are optional.

If `--nopersist` is specified, `mount(8)` is used instead of `appjail-fstab(1)`, that is, the mount point will not persist on reboot and must be unmounted before the jail is stopped.

By default, `appjail-fstab(1)` entries are compiled and mounted unless `--nomount` is specified. This option is recommended when you specify multiple entries: you can gain performance by specifying this option except for the last entry.

You can specify the identifier using `--nro`, but it is recommended to keep it as is, that is, `auto`, which is the default value.

### PKG

```
PKG [--chroot|--jexec [--jail]|--local] package ...
PKG [--chroot|--jexec [--jail]|--local] --remove package ...
PKG [--chroot|--jexec [--jail]|--local] --autoremove
PKG [--chroot|--jexec [--jail]|--local] --clean
PKG [--chroot|--jexec [--jail]|--local] --update
PKG [--chroot|--jexec [--jail]|--local] --upgrade
```

Manipulate packages.

`--chroot` is equivalent to `appjail-pkg(1)` `chroot`. This option can only be used for thick and thin jails, but the latter requires the jail to be started.

`--jexec` (**default**) is equivalent to `appjail-pkg(1)` `jail`. `--jail` is equivalent to `appjail-pkg(1)` `jail` `-j`.

`--local`, run `pkg(8)` on the host instead of inside the jail.

`--remove`, removes one or more packages instead of installing them.

`--autoremove`, removes orphaned or unused packages.

`--clean`, clean the local cache of fetched remote packages.

`--update`, update the package list.

`--upgrade`, perform upgrades of package software distributions.

### RAW

```
RAW [code]
```

Remember that an **InitScript** is `sh(1)` code and is generated by Makejails, so in many cases it is very useful for writing code that is processed as is, such as conditionals, loops, etc., however some instructions cannot be used for these purposes. See [Non-Conditional Instructions](#non-conditional-instructions).

### REPLACE

```
REPLACE file keyword [value] [output]
```

Replace a given <ins>keyword</ins> (without being enclosed in **%{** and **}**) with a <ins>value</ins> (or empty, if not defined) in a <ins>file</ins>. Keywords begin with the **%** character and then the keyword name enclosed in curly braces. Use **%** twice to escape, for example **%%{KEYWORD}** will be converted to **%{KEYWORD}**, but will not be replaced by any value. A different file can be used as <ins>output</ins> for the replaced keywords.

### RUN

```
RUN [--maintain-env] [--noclean] [--juser username|--huser username]
        [command [args ...]]
```

The `RUN` instruction executes a program, but unlike `CMD`, it cannot execute `sh(1)` code, it cannot execute interactive programs like Python, it cannot use variables created by `ARG` or `VAR` but it can use environment variables created by `ENV`, and instructions such as `USER`, `WORKDIR`, and `ENTRYPOINT` affect this instruction.

If `--maintain-env` is specified, leave the environment unchanged instead of simulating a full login.

The rest of the parameters have the same meaning as `CMD` `--jexec`.

### SERVICE

```
SERVICE args ...
```

Manipulate services. See `appjail-service(1)`.

### SET

```
SET [--mark] [--column column] [--row row] parameter[=value]
```

Use `appjail-config(1)` to edit the template used by the jail.

If `--mark` is specified, the given parameter is marked as required.

`--column` and `--row` can be specified to edit a specific parameter; However, if `--column` is set to a number greater than **1**, `appjail-config(1)` `setColumn` is used instead of `appjail-config(1)` `set`.

### STAGE

```
STAGE stage
```

Change the current stage. The default stage is **build**.

### SYSRC

```
SYSRC [--jail|--local] name[[+|-]=value] ...
```

Safely edit system rc files within a jail.

`--jail` is equivalent to `appjail-sysrc(1)` `jail`.

`--local` is equivalent to `appjail-sysrc(1)` `local`. It is only recommended to use this parameter with thick jails instead of thin jails, as it may not work correctly with the latter.

### UMOUNT

```
UMOUNT mountpoint
```

Unmount a mounted file system.

### USER

```
USER user
```

The user to run `RUN` as.

Unlike other instructions, this one cannot use shell variables.

### VAR

```
VAR [--make-arg-env] [--noexpand] name[=default-value]
```

Create or set a variable.

If `--make-arg-env` is specified, the variable is available to `CMD`.

If `--noexpand` the **$** character is escaped. Useful for build arguments.

### WORKDIR

```
WORKDIR directory
```

Create a new directory and use it as the working directory by `ADD`, `COPY`, and `RUN`.

Unlike other instructions, this one cannot use shell variables.

### LOGIN

```
LOGIN [--user username]
```

Log into the jail.

`--user` is equivalent to `appjail-login(1)` `-u`.

### OPTION

```
OPTION option
```

`appjail-quick(1)`'s options.

You don't need the option value enclosed in double quotes even when it has spaces.

### RESTART

```
RESTART
```

Restart the jail using `appjail-restart(1)`.

### START

```
START
```

### STOP

```
STOP
```

### VOLUME

```
VOLUME [--group gid] [--mountpoint mountpoint] [--owner owner] [--perm
        mode] [--type type] volume
```

Create a new volume.

`--group` is equivalent to `appjail-volume(1)` `add` `-g`.

`--mountpoint` is equivalent to `appjail-volume(1)` `add` `-m`.

`--owner` is equivalent to `appjail-volume(1)` `add` `-o`.

`--perm` is equivalent to `appjail-volume(1)` `add` `-p`.

`--type` is equivalent to `appjail-volume(1)` `add` `-t`.

## Non-Conditional Instructions

The following instructions cannot be used conditionally because they change the behavior of the resulting **InitScript** or **BuildScript** and do not generate code, or generate code that can only be used on specific lines:

* `ARG`
* `CLEAR`
* `ENTRYPOINT`
* `ENV`
* `GLOBAL`
* `INCLUDE`
* `STAGE`
* `USER`
* `VAR`: This instruction generates code that can be used conditionally, but if you use `--make-arg-env`, there is a side effect: even if you use this instruction conditionally, the environment variables will be available to `CMD`.
* `OPTION`

## Instructions that do not use the Tokenizer

The following instructions do not use the tokenizer, so they are parsed using their own methods:

* `ARG`
* `ENV`
* `ENTRYPOINT`
* `GLOBAL`
* `RAW`
* `USER`
* `UMOUNT`
* `WORKDIR`
* `OPTION`

## Build Stage Instructions

The following instructions are valid only in the **build** stage:

* `FROM`
* `LOGIN`
* `OPTION`
* `RESTART`
* `START`
* `STOP`

## Environment

* `APPJAIL_PWD`: `appjail-makejail(1)`, when processing the `INCLUDE` instruction, changes the current directory, so `PWD` does not reflect the current directory. Only available in the **build** stage.
