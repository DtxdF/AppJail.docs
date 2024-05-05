!!! tip

    Now an unprivileged user can run `appjail` without running `appjail-user` and this
    is the recommended way. Much of the following explanation actually applies.

When you share a server with co-workers or when you are the only person using a laptop, it is probably worth using AppJail without accessing the `root` account. AppJail has a simple but useful wrapper for such users named `appjail-user`.

The `appjail-user` uses `RUNAS` to execute AppJail commands as root. You can set it in the AppJail configuration file to whatever you prefer, such as `sudo` or `doas`. Of course, you need to install one of them first. I recommend using `security/doas` because it is simple and secure.

The only rule required in your `doas.conf(5)` file is:

```
permit nopass :appjail as root cmd appjail

# If you plan to use x11 applications, it is probably necessary to pass `keepenv`:
#permit nopass keepenv :appjail as root cmd appjail
```

If you want, you can remove `nopass` to require a password. This rule also assumes that you have a group named `appjail`. If you don't, don't worry:

```sh
pw groupadd -n appjail
```

To add your user to the `appjail` group simply run the following:

```sh
pw groupmod -n appjail -m "$USER"
```

Where `$USER` is your user. For these changes to take effect, you must log back into the system if you are adding yourself.

Now, any user that is in that group can run `appjail-user` as the administrator runs `appjail`:

```console
$ appjail-user jail list
```

Similarly, there is a variant for `appjail-config` named `appjail-config-user`. The instructions for using it are similar to the above:

```
permit nopass :appjail as root cmd appjail-config
```

Now, any user that is in that group can run `appjail-config-user` as the administrator runs `appjail-config`:

```console
$ appjail-config-user set -j myjail devfs_ruleset=15
```

Of course, unlike `appjail`, `appjail-config` does not require privileges for simple tasks like reading templates, but it does require privileges for writing them.
