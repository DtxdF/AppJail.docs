The Open Container Initiative (OCI) is an effort to create an open industry standard around container formats and runtimes. AppJail can interpret an OCI image to deploy the container using FreeBSD jails.

```console
# mkdir -p srv database config
# touch config/database.db
# appjail oci run \
    -d \
    -o overwrite=force \
    -o virtualnet=":<random> default" \
    -o nat \
    -o template=/usr/local/share/examples/appjail/templates/freebsd-oci.conf \
    -o fstab="$PWD/srv /srv" \
    -o fstab="$PWD/database/database.db database.db <pseudofs>" \
    -o fstab="$PWD/config/settings.json /.filebrowser.json <pseudofs>:reverse" \
    -e FB_NOAUTH=1 \
    docker.io/dtxdf007/filebrowser filebrowser
...
[00:00:04] [ debug ] [filebrowser] Creating a container (name:appjail-d11dccc9113) from docker.io/dtxdf007/filebrowser ...
appjail-d11dccc9113
...
[00:00:53] [ debug ] [filebrowser] Inspecting config.conf:
[00:00:53] [ debug ] [filebrowser]     appjail_version: 3.5.0+cf99039fea3f622e6b908803485be77527755323
[00:00:53] [ debug ] [filebrowser]     birth: 1733694151
[00:00:53] [ debug ] [filebrowser]     jail_type: thick
[00:00:53] [ debug ] [filebrowser]     release_name: default
[00:00:53] [ debug ] [filebrowser]     osarch: amd64
[00:00:53] [ debug ] [filebrowser]     osversion: 14.2-RELEASE
[00:00:53] [ debug ] [filebrowser]     container: 1
[00:00:53] [ debug ] [filebrowser]     container_image: docker.io/dtxdf007/filebrowser
...
[00:01:07] [ info  ] [filebrowser] Detached: pid:69856, log:jails/filebrowser/container/2024-12-08.log
```

With a single command we have created a jail and a container that has `filebrowser` installed. This command apart from creating the jail and the container, executes the command specified by the OCI image, and as we have specified the `-d` parameter, the process runs in the background.

```console
# env PAGER="cat" appjail logs read jails/filebrowser/container/2024-12-08.log
2024/12/08 18:08:09 Using database: /database.db
2024/12/08 18:08:09 Using config file: /.filebrowser.json
2024/12/08 18:08:09 Listening on [::]:80
```

If we stop and start the jail again, the process will not start because we have not specified the `-o "container=boot"` option, however, it is preferable to use this option with `appjail-oci(1)` `from` because the mentioned option will start the process and the `appjail-oci(1)` `run` command will perform the same task resulting in an error because only one background process can be executed per jail.

You can instruct the `appjail-start(1)` command to start the process in background using `appjail-oci(1)` ` `set-boot` `on`.

```sh
appjail oci set-boot on filebrowser
```

OCI containers expect to be configured through environment variables. You can specify to `appjail-start(1)` to use specific environment variables, so that the process can use them to suit your needs.

```sh
appjail oci set-env filebrowser FB_NOAUTH=1
```

`appjail-oci(1)` `run` will create a new jail and a new container each time. Maybe you just want to run a command to a container instead of creating a new one, the good news is that this command is just a wrapper to `appjail-oci(1)` `from` and `appjail-oci(1)` `exec`. We can use the latter command to perform this task.

```sh
appjail oci exec filebrowser sh
```

There are some useful keywords implemented in `appjail-jail(1)` `get` (and therefore in `appjail-jail(1)` `list`) that help manage our containers created by AppJail.

```console
# appjail jail list is_container container container_image container_pid container_boot name
IS_CONTAINER  CONTAINER            CONTAINER_IMAGE                 CONTAINER_PID  CONTAINER_BOOT  NAME
1             appjail-d11dccc9113  docker.io/dtxdf007/filebrowser  51722          1               filebrowser
```

---

**See also**:

* [Virtual Networks](networking/virtual-networks/intro.md)
* [NAT](networking/virtual-networks/NAT.md)
* [Templates](templates.md)
* [File System Management](fs-mgmt.md)
