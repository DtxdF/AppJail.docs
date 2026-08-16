!!! note

    Historically, there were tools called `appjail-user` and `appjail-config-user` that served as wrappers for `appjail` and `appjail-config` to elevate privileges. Now an unprivileged user can run `appjail`/`appjail-config` without running `appjail-user`/`appjail-config-user` and this is the recommended way.

When you share a server with co-workers or when you are the only person using a laptop, it is probably worth using AppJail without accessing the `root` account. `appjail` uses `RUNAS` (see `appjail.conf(5)`) to execute AppJail commands as `root`. You can set it in the AppJail configuration file to whatever you prefer, such as `sudo`, `doas` or native `mdo`. Of course, you need to install one of them first (except for `mdo`, which is native in FreeBSD). For simplicity, `RUNAS` defaults to `doas`, so the following document assumes `security/doas` is already installed on your system.

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

Now, any user that is in that group can run `appjail`.

```console
$ appjail jail list
```

For `appjail-config`, the instructions for using it are similar to the above:

```
permit nopass :appjail as root cmd appjail-config
```

Now, any user that is in that group can run `appjail-config`.

```console
$ appjail-config set -j myjail devfs_ruleset=15
```
