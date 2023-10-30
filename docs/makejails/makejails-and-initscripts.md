One problem that can occur when using a custom initscript with the `initscript` option and using Makejails is that the `start`, `stop` or `create` stages cannot be executed unless we are a little careful.

**initscript**:
```sh
start()
{
	echo "start::initscript"
}

stop()
{
	echo "stop::initscript"
}
```

**Makejail**:

```
OPTION overwrite
OPTION start
OPTION initscript=initscript

STAGE start

CMD --local echo "start::makejail"

STAGE stop

CMD --local echo "stop::makejail"
```

```console
# appjail makejail -f Makejail -j testinitscript
...
[00:00:34] [ debug ] [testinitscript] Running initscript `/usr/local/appjail/jails/testinitscript/init` ...
start::initscript
[00:00:34] [ debug ] [testinitscript] start() exits with status code 0
[00:00:34] [ debug ] [testinitscript] `/usr/local/appjail/jails/testinitscript/init` exits with status code 0
...
# appjail stop testinitscript
[00:00:02] [ debug ] [testinitscript] Running initscript `/usr/local/appjail/jails/testinitscript/init` ...
stop::makejail
[00:00:02] [ debug ] [testinitscript] stop() exits with status code 0
[00:00:02] [ debug ] [testinitscript] `/usr/local/appjail/jails/testinitscript/init` exits with status code 0
...
```

The `start` stage of the initscript has been executed because we used the `start` option. The `stop` stage of the initscript has not been executed because we did not use the `STOP` command in the `build` stage.

If you need to use a custom initscript, remember that it will be overwritten after the execution of the `buildscript` (see `Getting started with Makejail`), so use it only to build processes and remember to execute the stages using `START` or `OPTION start` and `STOP`.
