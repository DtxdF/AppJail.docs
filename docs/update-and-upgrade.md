Getting security patches and upgrading to a new version is very important and AppJail can do it with simple commands.

## update

A jail can be updated using `appjail-update(1)` `jail`:

```sh
appjail update jail blue
```

A version can be updated using `appjail-update(1)` `release` and if no parameters are used it uses the default variables specified in the configuration file.

```sh
# Update using default variables
appjail update release
# Update 12.3-RELEASE
appjail update release -v 12.3-RELEASE
```

## upgrade

Upgrading is a bit more complicated than just getting security patches, but in AppJail the process is simple.

Suppose the `apache2` jail is running on a `12.3-RELEASE` release and we want to upgrade it to `13.1-RELEASE`.

```sh
env PAGER=cat appjail upgrade jail -u -n 13.1-RELEASE apache2
```

The installation process should be executed as many times as necessary following the recommendations of the command.

```sh
env PAGER=cat appjail upgrade jail -i apache2
```

Upgrading a release is a little different than upgrade a jail. When AppJail upgrades a release, a copy is made and this directory is used.

The motivation for making a new copy is to preserve this directory for thinjail that are still using that release.

Another reason is to preserve this release, although you can destroy it using `appjail-fetch(1)` `destroy`.

To upgrade a release from `12.3-RELEASE` to `13.1-RELEASE`:

```sh
env PAGER=cat appjail upgrade release -u -n 13.1-RELEASE -v 12.3-RELEASE
env PAGER=cat appjail upgrade release -i -n 13.1-RELEASE -v 12.3-RELEASE
```

## Upgrading a thinjail

Thinjails are very useful, they save a lot of time and space and AppJail uses them by default. However, AppJail does not upgrade thinjails.

Instead of trying to upgrade, I recommend separating the data that the jail uses to function properly and the used data that must persist, and when a new version of FreeBSD arrives, just create a new jail with that version.

The data used by the jail is just stuff that can be obtained for each release, such as the base system programs, the configuration files used by those programs, etc. The data that must persist is such as a database, logs, custom applications, configuration files used by those applications or that you have modified, etc.

If you don't want to treat jails like cattle, use the pet version: thickjails.

See `appjail-ephemeral(7)` for details.

## Updating from a source tree

See [Source Tree](source-tree.md) for more information on updating a release and a jail using FreeBSD source tree.
