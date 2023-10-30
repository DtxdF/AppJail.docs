Initscripts are another useful AppJail feature that are the core of Makejail. An initscript is simply a `sh(1)` script that AppJail loads to run a `stage`. A `stage` is a function within the initscript that is executed by an AppJail command as follows:

* `create`: `appjail start` will run this `stage` before the jail starts.
* `start`: `appjail start` will run this `stage` after the jail starts.
* `stop`: `appjail stop` will run this `stage` after the jail is stopped.
* `cmd`: `appjail run` will run this `stage` only when the jail is running.
* `custom:<custom stage>`: `appjail run` will run this `stage` only when the jail is running. This stage is special because you can use any name you want (of course, not all characters are allowed).
* `apply`: `appjail apply` will run this `stage` only when the jail exists.

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

We use the `initscript` option in `appjail quick` to use this initscript in our new jail.

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

`appjail run` will run `cmd` whenever we want and only if the jail is running.

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

`appjail enable` is a command to enable arguments that need to be passed when the user does not provide them. This is necessary for commands such as `appjail startup` or `appjail restart` because these commands does not accept arguments for stages.

```sh
appjail enable myjail start -c 'create_msg=Hi everyone!' -s 'start_msg=Welcome.'
appjail enable myjail stop -p 'stop_msg=Bye.'
```

If we start or stop the jail without passing arguments, `appjail start` or `appjail stop` will use the arguments of the `appjail enable` command.

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

Initscripts are executed in the host, not in the jail. This decision is to run tasks in the host and tasks in the jail. To execute commands in a jail we use `jexec(8)`, but if we use fixed strings carelessly we may have some problems.

One problem we can see is that we rename a jail. The problem is that a command such as `jexec(8)` that uses the name of the jail to execute commands on it, will not execute correctly because the jail does not exist. Worse, there is a possibility that commands will be executed on a jail with totally different name than the one we intended.

AppJail solves this problem by keeping things simple: using the following environment variables:

* `APPJAIL_CONFIG`: AppJail configuration file.
* `APPJAIL_JAILDIR`: Jail directory (`{APPJAIL_ROOTDIR}/jail`)
* `APPJAIL_JAILNAME`: Jail name.
* `APPJAIL_ROOTDIR`: Root directory of the jail (`{JAILDIR}/{jail_name}`).
* `APPJAIL_SCRIPT`: AppJail script.
* `APPJAIL_PWD`: Since Makejail changes the current directory, `PWD` does not reflect the user's current directory. This environment variable allows a buildscript to know the user's current directory. Only available in the `build` stage of Makejail.

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

The above initscript has three custom stages: `python`, `php` and `top`. To run any of them, use `appjail run` with the `-s` parameter.

```sh
# python
appjail run -s python pyapp
# php
appjail run -s php pyapp
# top
appjail run -s top pyapp
```

Initscripts are a bit complex and there are some ways to create them much easier (See [Makejails](makejails/intro.md)).
