AppJail cannot parse a `jail.conf(5)` file, but it can parse its own format: templates.

The main motivation for using templates is to easily parse a file for scripts and for humans. See the following example:

```console
# appjail-config getAll -j nginx
exec.start: "/bin/sh /etc/rc"
exec.stop: "/bin/sh /etc/rc.shutdown jail"
mount.devfs
vnet
vnet.interface+: "eb_nginx"
exec.prestart+: "appjail network plug -e \"nginx\" -n \"web\""
exec.poststart+: "appjail network assign -d -e \"nginx\" -j \"${name}\" -n \"web\""
exec.poststop+: "appjail network unplug \"web\" \"nginx\""
exec.prestart+: "appjail nat on jail \"${name}\""
exec.poststop+: "appjail nat off jail \"${name}\""
exec.prestart+: "appjail expose on \"${name}\""
exec.poststop+: "appjail expose off \"${name}\""
exec.created+: "appjail limits on ${name}"
exec.poststop+: "appjail limits off ${name}"
```

The syntax is based on `jail.conf(5)`, but has some differences:

* Semicolon (`;`) separator is not used. AppJail parses the file line by line.
* `:` instead of `=`.
* `+:` instead of `+=`.
* Lists does not exist, but the AppJail tokenizer can achieve the same effect using rows and columns:
  - Rows are parameters that are repeated in the template. They are usually separated using `+:` when there is more than one, but it is optional, although it is very important to put the correct operator because AppJail translates the template to a `jail.conf(5)` file. A particular row can be accessed using an index from `0`.
  - Columns are the value of a row. They are separated using spaces, but the column in quotes (single or double) can be used to use spaces in the column. The `\` character must be used to escape `"` or `'`. A particular column can be accessed using an index from `0`.

AppJail tries to not to lose functionality by using this format. You can use variables, for example.

Although you can use any parameter you want in a template, `appjail start` intercepts some parameters:

* `exec.consolelog`: If not set, AppJail sets it in a console log type. See [Logs](logs.md).
* `mount.fstab`: If not set, AppJail uses the compiled fields of the `appjail fstab` command. See [File System Management](fs-mgmt.md).
* `host.hostname`: If not set, AppJail concatenates the jail name and `HOST_DOMAIN` (default: `.appjail`) defined in your AppJail configuration file.
* `depend`: If set, AppJail will recursively start these jails. See [Dependent Jails](dependent-jails.md).

As mentioned above, you can use rows and columns instead of lists. To illustrate this, the following is useful:

**example.conf**:

```console
exec.start: "/bin/sh /etc/rc"
exec.stop: "/bin/sh /etc/rc.shutdown jail"
mount.devfs
interface: jext
ip4.addr: 192.168.1.123/24 192.168.1.128/24
ip4.addr+: 10.42.0.4/10
```

The template has it all for learning about templates. For example, `exec.start` has spaces. `mount.devfs` has no value. `ip4.addr` has two rows. The first row of `ip4.addr` has two columns and the second row has one.

We can get the value of `exec.start`.

```console
# appjail-config get -t example.conf exec.start
exec.start: "/bin/sh /etc/rc"
```

The values are not escaped, since we are getting the whole value, not a column. To get the value of column `0` we can use:

```console
# appjail-config getColumn -t example.conf exec.start
/bin/sh /etc/rc
```

To get only the value of `exec.start` but not its parameter name:

```console
# appjail-config get -nt example.conf exec.start
"/bin/sh /etc/rc"
```

Another useful example is `ip4.addr`. To edit the whole value of row `1`:

```console
# appjail-config set -r 1 -t example.conf ip4.addr=10.42.0.3/10
# appjail-config get -r 1 -t example.conf ip4.addr
ip4.addr+: 10.42.0.3/10
```

However, if we want to edit row `0` with some columns, we probably do not want to edit the whole value. To edit only column `1`.

```console
# appjail-config setColumn -c 1 -t example.conf ip4.addr=192.168.1.176/24
# appjail-config get -t example.conf ip4.addr
ip4.addr: 192.168.1.123/24 192.168.1.176/24
```

Templates have another useful feature: `required parameters`.

Instead of using static parameters with values that may not be portable across multiple environments, we can use `required parameters` to force the user to edit some parameters. If the user does not edit those parameters, the jail won't start.

`required parameters` are parameters starting with an asterisk (`*`). The value is optional, but if set, `appjail start` uses it to display a custom message.

```
exec.start: "/bin/sh /etc/rc"
exec.stop: "/bin/sh /etc/rc.shutdown jail"
mount.devfs
vnet
*vnet.interface: VNET requires an interface.
```

!!! tip

    We can open an editor using `appjail-config edit` or use `appjail-config set -R1`
    to convert a parameter into a required one.

As mentioned, if we don't edit the template before starting the jail, `appjail start` will complain:

```console
# appjail quick vjail start template=/tmp/vnet.conf
...
[00:00:15] [ warn  ] [vjail] There are required parameters that must be set before starting the jail:
[00:00:15] [ warn  ] [vjail]     - vnet.interface: VNET requires an interface.
[00:00:15] [ warn  ] [vjail] You can use `appjail-config` to set the required parameters
```

We can use `appjail-config` to solve this problem. We can use `appjail-config edit` to open the editor specified by the `EDITOR` environment variable or use `appjail-config set` as if we were configuring any other parameter.

```sh
# Using ${EDITOR}.
appjail-config edit -j vjail
# Using command-line interface.
appjail-config set -j vjail vnet.interface=jext
```

To see the change we can use `appjail-config get`.

```console
# appjail-config get -j vjail vnet.interface
vnet.interface: jext
```

If you want to list the `required parameters`:

```console
# appjail-config getAll -rt alias.conf
*interface: Interface name.
*ip4.addr: IPv4 address to use.
```

**alias.conf**:

```
exec.start: "/bin/sh /etc/rc"
exec.stop: "/bin/sh /etc/rc.shutdown jail"
mount.devfs
*interface: Interface name.
*ip4.addr: IPv4 address to use.
```

`appjail-config` has many parameters to list here, but I think the above examples show the basics. Use `appjail-config help` and `appjail-config help [cmd]` to get more details.
