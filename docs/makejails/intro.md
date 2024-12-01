Makejail is a simple text file that automates the steps of creating a jail, by generating an **InitScript** and a **BuildScript**.

Makejails are processed line by line, removing comments, empty lines and including the Makejails specified by `INCLUDE` (if any) in a single, temporary Makejail that is responsible for executing the rest of the supported instructions. See `INCLUDE` for more details.

Makejail files are divided in stages that can be changed using the STAGE instruction, which are actually functions as described in `appjail-initscript(5)`, except the build stage which is not used by **InitScript**, but is used by the **BuildScript**. See `appjail-makejail(1)` for more details.

Let's create our first Makejail:

```
STAGE cmd

CMD echo "Hello, world!"
```

To run this Makejail, use `appjail-makejail(1)`.

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
