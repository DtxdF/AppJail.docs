A Makejail file is a sequence of instructions, line by line, where each command and its arguments are separated by a single space (ascii: `0x20`).

As you can see in `INCLUDE`, a lot of useless information such as leading spaces, comments, empty lines and other similar things are discarded.

When `INCLUDE` compiles all Makejails into a single Makejail, `appjail makejail` uses it to execute the instructions.

The first stage to execute is `build`. This stage writes a script named `buildscript` that takes care of building the jail.

When `buildscript` successfully completes its execution, the `initscript` is written to the jail directory, overwriting another `initscript` (if any).

`appjail makejail` executes the commands in certain order. First, the `ARG` command is executed, second, `OPTION` (build only), and the rest ot the commands are executed in the order they appear.

Now, we can write a Makejail.

```
STAGE cmd

CMD echo "Hello, world!"
```

To run this Makejail, use `appjail makejail`.

```sh
appjail makejail -f hello.makejail -j hello
```

Remember that the `-f` parameter can use the methods described in `INCLUDE` and if you do not use a name for the jail with `-j`, a random name is chosen.

```console
# appjail start hello
...
# appjail run hello
[00:00:02] [ debug ] [hello] Running initscript `/usr/local/appjail/jails/hello/init` ...
Hello, world!
[00:00:03] [ debug ] [hello] cmd() exits with status code 0
[00:00:03] [ debug ] [hello] `/usr/local/appjail/jails/hello/init` exits with status code 0
```
