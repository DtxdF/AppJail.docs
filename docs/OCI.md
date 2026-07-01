The Open Container Initiative (OCI) is an effort to create an open industry standard around container formats, runtimes, and distribution.

Thanks to the OCI, we can create an image that many tools can deploy, likely in an isolated environment such as a FreeBSD jail. The only requisite is that the tool be OCI-compatible, and, at least in AppJail, this compatibility is very well implemented, allowing OCI containers to be deployed quickly using specialized commands.

!!! info

    [AppJail images](images/intro.md) should not be confused with OCI images. AppJail images were implemented because, at least on FreeBSD, OCI had not yet been implemented at that time, and AppJail images were the only viable option for creating snapshots of the jails in a binary format that allowed for easy distribution.

Let's deploy a simple web application.

```console
# mkdir -p .volumes/navidrome .volumes/music
# appjail oci run -Pd \
  -o overwrite=force \
  -o container="args:--pull" \
  -o virtualnet=":<random> default" \
  -o nat \
  -o expose="4533" \
  -e PUID=15000 \
  -e PGID=15000 \
  -e TZ=America/Caracas \
  -e ND_SCANNER_SCHEDULE="@every 1h" \
  -e ND_LOGLEVEL=info \
  -o fstab="$PWD/.volumes/navidrome /config <pseudofs>" \
  -o fstab="$PWD/.volumes/music /music <pseudofs>" \
  ghcr.io/daemonless/navidrome:latest navidrome
[00:02:10] [ info  ] [navidrome] Detached: pid:33410, log:jails/navidrome/container/2026-06-24.log
# appjail jail list -j navidrome
STATUS  NAME       ALT_NAME  TYPE   VERSION       PORTS     NETWORK_IP4
UP      navidrome  -         thick  15.0-RELEASE  4533/tcp  10.0.0.8
# appjail jail list -j navidrome name container_pid
NAME       CONTAINER_PID
navidrome  33410
# appjail logs tail jails/navidrome/container/2026-06-24.log -f
time="2026-06-24T00:49:43-04:00" level=info msg="Mounting WebUI routes" path=/app
time="2026-06-24T00:49:43-04:00" level=info msg="Creating backgrounds cache" maxSize="100 MB" path=/config/data/cache/backgrounds
time="2026-06-24T00:49:43-04:00" level=info msg="Finished initializing cache" cache=backgrounds elapsedTime="242.436µs" maxSize=100MB
time="2026-06-24T00:49:43-04:00" level=info msg="----> Navidrome server is ready!" address="0.0.0.0:4533" startupTime=963.5ms tlsEnabled=false
time="2026-06-24T00:49:45-04:00" level=warning msg="Full scan required after migration"
time="2026-06-24T00:49:45-04:00" level=info msg="Loaded configuration" file=/config/config.toml
time="2026-06-24T00:49:45-04:00" level=info msg="Scanner: Starting scan" fullScan=true numLibraries=1
time="2026-06-24T00:49:45-04:00" level=warning msg="Playlists will not be imported, as there are no admin users yet, Please create an admin user first, and then update the playlists for them to be imported"
time="2026-06-24T00:49:45-04:00" level=info msg="Scanner: Finished scanning all libraries" duration=8ms
time="2026-06-24T00:49:45-04:00" level=info msg="Scan completed"
```

Let's break down each parameter:

1. `-P`: The environment variables, working directory, and user you have specified will be preserved, which means that if you restart the jail, AppJail will use the parameters you defined in this command. If the OCI image provides a command to be executed, it will also run in the background, even if the jail is restarted.
2. `-d`: The process will run in the background.
3. `-o overwrite=force`: Destroy the jail if it already exists, so that AppJail will recreate it instead of refusing to do so.
4. `-o container="args:--pull"`: Let's pull the image every time `buildah(1)` detects changes, so that AppJail always runs the jail using the latest image.
5. `-o virtualnet=":<random> default" -o nat -o expose="4533"`: Network options. In this case, we chose to use [Virtual Networks](networking/virtual-networks/intro.md). And creating a port mapping for port `4533` may be unnecessary if you do not want external clients to communicate with your service and prefer to use the one assigned by the virtual network (in this case, `10.0.0.8`).
6. `-e PUID=15000 -e PGID=15000 -e TZ=America/Caracas -e ND_SCANNER_SCHEDULE="@every 1h" -e ND_LOGLEVEL=info`: Environment variables specific to this OCI image. This depends entirely on the image and our preferences.
7. `-o fstab="$PWD/.volumes/navidrome /config <pseudofs>" -o fstab="$PWD/.volumes/music /music <pseudofs>"`: Volumes ensure that data persists even if the container is destroyed or recreated.
8. `ghcr.io/daemonless/navidrome:latest navidrome`: The image, tag, and the jail name. The tag is optional.

With a single command, we have an OCI container deployed and ready to use. However, it's worth mentioning that most options depend entirely on our preferences, the OCI image we want to use, and even the process to be run.

!!! warning

    You need to install `sysutils/buildah` and `textproc/jq` before using the `appjail-oci(1)` command.

### `Containerfile(5)`

The simplest and most recommended way to customize or create an OCI image is to use `Containerfile(5)`. For example, if you want to create a container from a PHP script:

**Containerfile**:

```dockerfile
FROM ghcr.io/appjail-makejails/php:15.1-85
COPY . /hello
WORKDIR /hello
CMD ["php", "./hello.php"]
```

**hello.php**:

```php
<?php

echo "Hello, world!\n";

?>
```

Then, run the commands to build and run the OCI image:

```console
# buildah build --network=host -t hello-php .
# appjail oci run \
    -o overwrite=force \
    -o ephemeral \
    -o alias \
    -o ip4_inherit \
    localhost/hello-php hello-php
...
Hello, world!
```

Check the [official documentation](https://github.com/AppJail-makejails/php) for this OCI image to see more examples.

### OCI and Makejails

If you prefer a more traditional approach, you can use Makejails to deploy an OCI container. A jail will be created from the OCI image (mounted or imported, depending on the options you’ve used) and then customized using the Makejail. For example, [Navidrome](https://github.com/AppJail-makejails/navidrome), from AppJail-makejails, can be deployed using either `appjail oci run` or `appjail makejail`. Let’s take a look at the current Makejail.

```
ARG navidrome_from=ghcr.io/appjail-makejails/navidrome
ARG navidrome_tag=latest

OPTION start
OPTION overwrite=force
OPTION from=${navidrome_from}:${navidrome_tag}
OPTION volume=navidrome-music mountpoint:/usr/local/share/navidrome/music owner:${puid} group:${pgid}
OPTION volume=navidrome-db mountpoint:/var/db/navidrome owner:${puid} group:${pgid}

INCLUDE gh+AppJail-makejails/user-mapping

CMD --local appjail oci set-user "${APPJAIL_JAILNAME}" noroot
CMD --local appjail oci set-boot on "${APPJAIL_JAILNAME}"
CMD chown -R noroot:noroot /var/db/navidrome

# The first run won't start the OCI process because we used the 'start'
# option, so a second run is required for this to happen. For Director
# users, this goes virtually unnoticed.
STOP
```

Combining a Makejail with an OCI image may seem overkill, but this combination allows you to get the best of both worlds. Specifically, Makejails let you customize the jail at runtime more easily.

In the previous example, the `from` option of `appjail-quick(1)` (don't confuse `from` from `appjail-quick(1)` with `FROM` from `appjail-makejail(5)`) mounts the OCI image from `ghcr.io/appjail-makejails/navidrome` using the `latest` tag. These values are derived from the arguments specified above (`navidrome_from` and `navidrome_tag`), so the user can modify them at runtime. For example, a user can build a custom OCI image and then reuse the same Makejail to deploy a jail. Later, we’ll see that some commands are run from the host to configure the jail itself so that the process runs inside it as `noroot` (using `appjail oci set-user`) and to set the boot option so that the process runs at boot time (using `appjail oci set-boot on`). This user is created from the Makejail we included earlier, [gh+AppJail-makejails/user-mapping](https://github.com/AppJail-makejails/user-mapping) which is created using a custom UID and GID specified at runtime.

Next, to deploy the above, we can use the following commands:

```console
# appjail makejail \
    -j navidrome \
    -f gh+AppJail-makejails/navidrome \
    -o virtualnet=":<random> default" \
    -o nat \
    -o expose="4533" \
    -o container="args:--pull"
# appjail start navidrome
```

---

**See also**:

* [Virtual Networks](networking/virtual-networks/intro.md)
* [NAT](networking/virtual-networks/NAT.md)
* [Templates](templates.md)
* [File System Management](fs-mgmt.md)
* [Images](images/intro.md)
