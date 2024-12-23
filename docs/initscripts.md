**InitScripts** are another useful AppJail feature that are the core of Makejail. An initscript is simply a `sh(1)` script that AppJail loads to run a `stage`. A `stage` is a function within the initscript that is executed by an AppJail command as follows:

* `create`: Stage executed by `appjail-start(1)` before the jail is started.
* `start`: Stage executed by `appjail-start(1)` after the jail is started.
* `stop`: Stage executed by `appjail-stop(1)` before the jail is stopped.
* `cmd`: Stage executed by `appjail-run(1)`.
* `custom:<arbitrary string>`: Stage executed by appjail-run(1). The difference between cmd and this stage is that the latter can be defined using a valid arbitrary string. A valid arbitrary string is `^[a-zA-Z0-9_][a-zA-Z0-9_-]*$`.
* `apply`: Stage executed by `appjail-apply(1)`.

Each `stage` has `pre-` and `post-` functions. `pre-` is executed before `stage` is executed and if it fails, `stage` will not be executed. `post-` will be executed after `pre-` and `stage` even if they fail. `pre-` and `stage` affect the exit status, but `post-` does not. `stage` and functions in a initscript are all optional, AppJail will execute them if they exist.

The following example contains all stages and functions:

```sh
_parse_args()
{
	local arg
	for arg in "$@"; do
		printf "<%s> " "${arg}"
	done
	echo
}

precreate()
{
	echo -n "precreate args: "
	_parse_args "$@"
}

create()
{
	echo -n "create args: "
	_parse_args "$@"
}

postcreate()
{
	echo -n "postcreate args: "
	_parse_args "$@"
}

prestart()
{
	echo -n "prestart args: "
	_parse_args "$@"
}

start()
{
	echo -n "start args: "
	_parse_args "$@"
}

poststart()
{
	echo -n "poststart args: "
	_parse_args "$@"
}

precmd()
{
	echo -n "precmd args: "
	_parse_args "$@"
}

cmd()
{
	echo -n "cmd args: "
	_parse_args "$@"
}

postcmd()
{
	echo -n "postcmd args: "
	_parse_args "$@"
}

prestop()
{
	echo -n "prestop args: "
	_parse_args "$@"
}

stop()
{
	echo -n "stop args: "
	_parse_args "$@"
}

poststop()
{
	echo -n "poststop args: "
	_parse_args "$@"
}
```

We use the `initscript` option in `appjail-quick(1)` to use this initscript in our new jail.

```sh
chmod +x /tmp/initscript
appjail quick myjail overwrite initscript=/tmp/initscript
```

The jail will not start because we are not using the `start` option, this is because we will start the jail manually to pass arguments for `create` and for `start` stages.

```console
# appjail start -c 'parameter1=I am the create parameter #1' -c 'parameter2=I am the create parameter #2' -s 'parameter1=I am the start parameter #1' -s 'parameter2=I am the start parameter #2' myjail
...
[00:00:17] [ debug ] [myjail] Running initscript `/usr/local/appjail/jails/myjail/init` ...
[00:00:17] [ debug ] [myjail] Running precreate() ...
precreate args: <--parameter1> <I am the create parameter #1> <--parameter2> <I am the create parameter #2>
[00:00:17] [ debug ] [myjail] precreate() exits with status code 0
create args: <--parameter1> <I am the create parameter #1> <--parameter2> <I am the create parameter #2>
[00:00:17] [ debug ] [myjail] create() exits with status code 0
postcreate args: <--parameter1> <I am the create parameter #1> <--parameter2> <I am the create parameter #2>
[00:00:17] [ debug ] [myjail] postcreate() exits with status code 0
[00:00:17] [ debug ] [myjail] `/usr/local/appjail/jails/myjail/init` exits with status code 0
[00:00:17] [ debug ] [myjail] Creating...
[00:00:19] [ info  ] [myjail] myjail: created
[00:00:20] [ debug ] [myjail] Running initscript `/usr/local/appjail/jails/myjail/init` ...
[00:00:20] [ debug ] [myjail] Running prestart() ...
prestart args: <--parameter1> <I am the start parameter #1> <--parameter2> <I am the start parameter #2>
[00:00:20] [ debug ] [myjail] prestart() exits with status code 0
start args: <--parameter1> <I am the start parameter #1> <--parameter2> <I am the start parameter #2>
[00:00:20] [ debug ] [myjail] start() exits with status code 0
poststart args: <--parameter1> <I am the start parameter #1> <--parameter2> <I am the start parameter #2>
[00:00:20] [ debug ] [myjail] poststart() exits with status code 0
[00:00:20] [ debug ] [myjail] `/usr/local/appjail/jails/myjail/init` exits with status code 0
```

As you can see, the `pre-` and `post-` functions receive the same parameters as their `stage`.

`appjail-run(1)` will run `cmd` whenever we want and only if the jail is running.

```console
# appjail run -p 'msg=Hello, world!' myjail
[00:00:01] [ debug ] [myjail] Running initscript `/usr/local/appjail/jails/myjail/init` ...
[00:00:01] [ debug ] [myjail] Running precmd() ...
precmd args: <--msg> <Hello, world!>
[00:00:01] [ debug ] [myjail] precmd() exits with status code 0
cmd args: <--msg> <Hello, world!>
[00:00:01] [ debug ] [myjail] cmd() exits with status code 0
postcmd args: <--msg> <Hello, world!>
[00:00:01] [ debug ] [myjail] postcmd() exits with status code 0
[00:00:01] [ debug ] [myjail] `/usr/local/appjail/jails/myjail/init` exits with status code 0
```

Finally, we can pass arguments to `stop`:

```console
# appjail stop -p 'msg=Bye ...' myjail
[00:00:02] [ debug ] [myjail] Running initscript `/usr/local/appjail/jails/myjail/init` ...
[00:00:02] [ debug ] [myjail] Running prestop() ...
prestop args: <--msg> <Bye ...>
[00:00:02] [ debug ] [myjail] prestop() exits with status code 0
stop args: <--msg> <Bye ...>
[00:00:02] [ debug ] [myjail] stop() exits with status code 0
poststop args: <--msg> <Bye ...>
[00:00:02] [ debug ] [myjail] poststop() exits with status code 0
[00:00:02] [ debug ] [myjail] `/usr/local/appjail/jails/myjail/init` exits with status code 0
...
```

`appjail-enable(1)` is a command to enable arguments that need to be passed when the user does not provide them. This is necessary for commands such as `appjail-startup(1)` or `appjail-restart(1)` because these commands does not accept arguments for stages.

```sh
appjail enable myjail start -c 'create_msg=Hi everyone!' -s 'start_msg=Welcome.'
appjail enable myjail stop -p 'stop_msg=Bye.'
```

If we start or stop the jail without passing arguments, `appjail-start(1)` or `appjail-stop(1)` will use the arguments of the `appjail-enable(1)` command.

```console
# appjail start myjail
...
[00:00:18] [ debug ] [myjail] Running initscript `/usr/local/appjail/jails/myjail/init` ...
[00:00:18] [ debug ] [myjail] Running precreate() ...
precreate args: <--create_msg> <Hi everyone!>
[00:00:18] [ debug ] [myjail] precreate() exits with status code 0
create args: <--create_msg> <Hi everyone!>
[00:00:18] [ debug ] [myjail] create() exits with status code 0
postcreate args: <--create_msg> <Hi everyone!>
[00:00:18] [ debug ] [myjail] postcreate() exits with status code 0
[00:00:18] [ debug ] [myjail] `/usr/local/appjail/jails/myjail/init` exits with status code 0
[00:00:18] [ debug ] [myjail] Creating...
[00:00:18] [ info  ] [myjail] myjail: created
[00:00:19] [ debug ] [myjail] Running initscript `/usr/local/appjail/jails/myjail/init` ...
[00:00:19] [ debug ] [myjail] Running prestart() ...
prestart args: <--start_msg> <Welcome.>
[00:00:19] [ debug ] [myjail] prestart() exits with status code 0
start args: <--start_msg> <Welcome.>
[00:00:19] [ debug ] [myjail] start() exits with status code 0
poststart args: <--start_msg> <Welcome.>
[00:00:19] [ debug ] [myjail] poststart() exits with status code 0
[00:00:19] [ debug ] [myjail] `/usr/local/appjail/jails/myjail/init` exits with status code 0
```

**InitScripts** are executed in the host, not in the jail. This decision is to run tasks in the host and tasks in the jail. To execute commands in a jail we use `jexec(8)`, but if we use fixed strings carelessly we may have some problems.

One problem we can see is that we rename a jail. The problem is that a command such as `jexec(8)` that uses the name of the jail to execute commands on it, will not execute correctly because the jail does not exist. Worse, there is a possibility that commands will be executed on a jail with totally different name than the one we intended.

AppJail solves this problem by keeping things simple: using the following environment variables:

* `APPJAIL_CONFIG`: AppJail configuration file.
* `APPJAIL_JAILDIR`: Jail directory (`{APPJAIL_ROOTDIR}/jail`)
* `APPJAIL_JAILNAME`: Jail name.
* `APPJAIL_ROOTDIR`: Root directory of the jail (`{JAILDIR}/{jail_name}`).
* `APPJAIL_SCRIPT`: AppJail script.

With the above information we can make an initscript to display `Hello, world!`:

```sh
cmd()
{
	jexec -l "${APPJAIL_JAILNAME}" sh -c 'echo "Hello, world!"'
}
```

```console
# appjail quick myjail initscript=/tmp/initscript start overwrite
# appjail run myjail
[00:00:06] [ debug ] [myjail] Running initscript `/usr/local/appjail/jails/myjail/init` ...
Hello, world!
[00:00:06] [ debug ] [myjail] cmd() exits with status code 0
[00:00:07] [ debug ] [myjail] `/usr/local/appjail/jails/myjail/init` exits with status code 0
```

As mentioned above, you can use a custom stage. This is very useful when many Makejails are included so as not to overlap stages.

```sh
custom:python()
{
        "${APPJAIL_SCRIPT}" cmd jexec "${APPJAIL_JAILNAME}" python3.9
}

custom:php()
{
        "${APPJAIL_SCRIPT}" cmd jexec "${APPJAIL_JAILNAME}" php -a
}

custom:top()
{
        "${APPJAIL_SCRIPT}" cmd jexec "${APPJAIL_JAILNAME}" top -a
}
```

The above initscript has three custom stages: `python`, `php` and `top`. To run any of them, use `appjail-run(1)` with the `-s` parameter.

```sh
# python
appjail run -s python pyapp
# php
appjail run -s php pyapp
# top
appjail run -s top pyapp
```

**InitScripts** are a bit complex and there are some ways to create them much easier (See [Makejails](makejails/intro.md)).

### InitScript Library

An **InitScript** is as powerful as any other `sh(1)` script, the problem is that although the built-in environment variables can help, they are not enough for some common tasks, that's why there is a "library" with some subroutines you can use, inspired by `appjail-makejail(5)`.

**initscript**:

```sh
. "${LIBDIR}/initscript"

create()
{
    local volume_created
    volume_created=`LABEL:GET initscripts.volumes.shared`

    if [ -z "${volume_created}" ]; then
        VOLUME -m /shared shared-dir

        LABEL:ADD initscripts.volumes.shared 1
    fi
}

start()
{
	ARG enable_bpf 0
	ARG install_htop 0
	ARG install_nginx 0
	ARG nginx_conf
	ARG nginx_worker_processes auto
	ARG nginx_worker_connections 1024
	ARG nginx_keepalive_timeout 65
	ARG nginx_server_name localhost

	PARSE "$@"

	[ -d /shared ] || mkdir -p /shared

	local shared_mounted
	shared_mounted=`LABEL:GET initscripts.mounted.shared`

	if [ -z "${shared_mounted}" ]; then
		MOUNT /shared shared-dir "<volumefs>" ro || exit $?

		LABEL:ADD initscripts.mounted.shared 1 || exit $?
	fi

	if [ "${ARG_enable_bpf}" != 0 ]; then
		local bpf
		bpf=`LABEL:GET initscripts.devices.bpf`

		if [ -z "${bpf}" ]; then
			DEVICE:SET 'include $devfsrules_hide_all' || exit $?
			DEVICE:SET 'include $devfsrules_unhide_basic' || exit $?
			DEVICE:SET 'include $devfsrules_unhide_login' || exit $?
			DEVICE:SET 'path "bpf*" unhide' || exit $?
			DEVICE:SET 'path bpf unhide' || exit $?
			DEVICE:APPLYSET || exit $?

			LABEL:ADD initscripts.devices.bpf 1 || exit $?
		fi
	fi

	local htop_installed
	htop_installed=`LABEL:GET initscripts.packages.htop`

	if [ "${ARG_install_htop}" != 0 ] && [ -z "${htop_installed}" ]; then
		PKG install -y htop || exit $?

		LABEL:ADD initscripts.packages.htop 1 || exit $?
	fi

	local nginx_installed
	nginx_installed=`LABEL:GET initscripts.packages.nginx`

	if [ "${ARG_install_nginx}" != 0 ] && [ -z "${nginx_installed}" ]; then
		PKG install -y nginx || exit $?

		if [ -n "${ARG_nginx_conf}" ]; then
			if [ ! -f "${ARG_nginx_conf}" ]; then
				printf "%s: nginx configuration file cannot be found.\n" "${ARG_nginx_conf}"
				return 1
			fi

			local nginx_conf
			nginx_conf="/usr/local/etc/nginx/nginx.conf"

			cp -af "${ARG_nginx_conf}" "${APPJAIL_JAILDIR}/${nginx_conf}" || exit $?

			REPLACE "${nginx_conf}" WORKER_PROCESSES "${ARG_nginx_worker_processes}" || exit $?
			REPLACE "${nginx_conf}" WORKER_CONNECTIONS "${ARG_nginx_worker_connections}" || exit $?
			REPLACE "${nginx_conf}" KEEPALIVE_TIMEOUT "${ARG_nginx_keepalive_timeout}" || exit $?
			REPLACE "${nginx_conf}" SERVER_NAME "${ARG_nginx_server_name}" || exit $?
		fi

		SYSRC nginx_enable=YES || exit $?
		SERVICE nginx start || exit $?

		LABEL:ADD initscripts.packages.nginx 1 || exit $?
	fi
}

cmd()
{
	if ! CMD which -s htop; then
		echo "sysutils/htop is not installed!" >&2
		return 1
	fi

	CMD htop
}

custom:test_pwd()
{
	WORKDIR /shared

	CMD pwd
}

custom:test_env()
{
	INITENV

	CMD env
}

custom:test_chroot()
{
	CMD /bin/sh
}

custom:test_jaildir()
{
	JAILDIR /bin/sh
}

custom:test_local()
{
	LOCAL /bin/sh
}

stop()
{
	echo "Good byte!"
}
```

The above file demonstrates how much simpler it is to use the library.

```console
# appjail quick jtest \
    start \
    overwrite=force \
    alias \
    ip4_inherit \
    start_args="install_nginx=1" \
    start_args="install_htop=1" \
    initscript="$PWD/initscript"
...
[00:00:30] [ debug ] [jtest] Compiling fstab #0: /shared shared-dir <volumefs> ro 0 0
Updating FreeBSD repository catalogue...
Fetching meta.conf: 100%    178 B   0.2kB/s    00:01
Fetching data.pkg: 100%    7 MiB 324.1kB/s    00:23
Processing entries: 100%
FreeBSD repository update completed. 35543 packages processed.
All repositories are up to date.
The following 1 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
        htop: 3.3.0_5

Number of packages to be installed: 1

107 KiB to be downloaded.
[1/1] Fetching htop-3.3.0_5.pkg: 100%  107 KiB 109.9kB/s    00:01
Checking integrity... done (0 conflicting)
[1/1] Installing htop-3.3.0_5...
[1/1] Extracting htop-3.3.0_5: 100%
Updating FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
The following 2 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
        nginx: 1.26.2_5,3
        pcre2: 10.43

Number of packages to be installed: 2

The process will require 9 MiB more space.
2 MiB to be downloaded.
[1/2] Fetching nginx-1.26.2_5,3.pkg: 100%  558 KiB 571.2kB/s    00:01
[2/2] Fetching pcre2-10.43.pkg: 100%    1 MiB 483.7kB/s    00:03
Checking integrity... done (0 conflicting)
[1/2] Installing pcre2-10.43...
[1/2] Extracting pcre2-10.43: 100%
[2/2] Installing nginx-1.26.2_5,3...
===> Creating groups
Using existing group 'www'
===> Creating users
Using existing user 'www'
[2/2] Extracting nginx-1.26.2_5,3: 100%
=====
Message from nginx-1.26.2_5,3:

--
Recent version of the NGINX introduces dynamic modules support.  In
FreeBSD ports tree this feature was enabled by default with the DSO
knob.  Several vendor's and third-party modules have been converted
to dynamic modules.  Unset the DSO knob builds an NGINX without
dynamic modules support.

To load a module at runtime, include the new `load_module'
directive in the main context, specifying the path to the shared
object file for the module, enclosed in quotation marks.  When you
reload the configuration or restart NGINX, the module is loaded in.
It is possible to specify a path relative to the source directory,
or a full path, please see
https://www.nginx.com/blog/dynamic-modules-nginx-1-9-11/ and
http://nginx.org/en/docs/ngx_core_module.html#load_module for
details.

Default path for the NGINX dynamic modules is

/usr/local/libexec/nginx.
nginx_enable:  -> YES
Performing sanity check on nginx configuration:
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful
Starting nginx.
[00:01:57] [ debug ] [jtest] start() exits with status code 0
[00:01:57] [ debug ] [jtest] `/usr/local/appjail/jails/jtest/init` exits with status code 0
[00:01:58] [ debug ] [jtest] Unlocking jtest ...
[00:01:59] [ debug ] [jtest] Done.
# appjail fstab jail jtest
NRO  ENABLED  NAME  DEVICE   MOUNTPOINT  TYPE        OPTIONS  DUMP  PASS
0    1        -     /shared  shared-dir  <volumefs>  ro       0     0
# appjail volume list jtest
NAME        MOUNTPOINT  TYPE        UID  GID  PERM
shared-dir  /shared     <pseudofs>  -    -    -
# appjail label list jtest
NAME                        VALUE
initscripts.mounted.shared  1
initscripts.packages.htop   1
initscripts.packages.nginx  1
# appjail run -s test_pwd jtest
[00:00:02] [ debug ] [jtest] Running initscript `/usr/local/appjail/jails/jtest/init` ...
/shared
[00:00:03] [ debug ] [jtest] custom:test_pwd() exits with status code 0
[00:00:03] [ debug ] [jtest] `/usr/local/appjail/jails/jtest/init` exits with status code 0
# appjail run -s test_env -V env1=1234 -V env2=4321 jtest
[00:00:02] [ debug ] [jtest] Running initscript `/usr/local/appjail/jails/jtest/init` ...
env2=4321
env1=1234
SHELL=/bin/sh
HOME=/root
USER=root
BLOCKSIZE=K
MAIL=/var/mail/root
MM_CHARSET=UTF-8
LANG=C.UTF-8
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/sbin:/usr/local/bin:/root/bin
TERM=screen.xterm-256color
[00:00:02] [ debug ] [jtest] custom:test_env() exits with status code 0
[00:00:02] [ debug ] [jtest] `/usr/local/appjail/jails/jtest/init` exits with status code 0
# appjail stop jtest
[00:00:03] [ debug ] [jtest] Running initscript `/usr/local/appjail/jails/jtest/init` ...
Good byte!
[00:00:03] [ debug ] [jtest] stop() exits with status code 0
[00:00:03] [ debug ] [jtest] `/usr/local/appjail/jails/jtest/init` exits with status code 0
[00:00:03] [ warn  ] [jtest] Stopping jtest...
jtest: removed
[00:00:05] [ debug ] [jtest] unmounting: umount "/usr/local/appjail/jails/jtest/jail/.appjail"
```

One of the most important parts of this initscript are the labels. Without labels, some functions will generate an error because they will not be able to run again. Or even when a function has not returned an error, such as `PKG`, it may be useful not to run it again.
