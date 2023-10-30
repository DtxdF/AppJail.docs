To create a very basic jail, only two things are needed: obtaining the FreeBSD components (`base.txz`, `lib32.txz`, etc.) and creating the jail using those components.

```console
appjail fetch
appjail quick myjail start login
```

Using the `appjail fetch` command will download the `MANIFEST` file to check the components. Afterwards, AppJail will download the components. By default, AppJail will only download `base.txz`. AppJail will extract those components into its release directory.

At this point, AppJail can create a jail using the `appjail quick` command. In the above example, `appjail quick` will create a jail named `myjail`. Using the `start` option, AppJail will start the jail after its creation. The `login` option simply logs into the jail after startup.

The `appjail fetch` is not necessary to run again unless you need another release with different components.

!!! tip

    AppJail is able to work without a configuration file, but it is highly recommended
    to [configure it](configure.md) for performance and reliability reasons.

!!! tip

    AppJail has a very useful command if you want to get more information about a command
    and its parameters called `appjail help`.
