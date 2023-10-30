TinyJails is a feature of AppJail to export a jail with only the files needed to run the application.

TinyJails is tricky because you need to know the application and its files.

This is an experimental feature and many things are not automated at this time.

We will use a `hello` program to illustrate how to use this feature.

```sh
cat << "EOF" > hello.c
#include <stdio.h>
#include <stdlib.h>

int
main(void)
{
printf("Hello, world!\n");
return EXIT_SUCCESS;
}
EOF
cc -o hello hello.c

appjail quick tinyjail
appjail cmd local tinyjail mkdir -p usr/local/bin
appjail cmd local tinyjail mv "$PWD/hello" usr/local/bin
cat << "EOF" > files.lst
/usr/local/bin/hello
EOF
appjail jail create -I tiny+export="files:files.lst output:hello.appjail compress:xz" tinyjail
```

AppJail checks all files to be copied to see if they point to a path outside of the jail directory.

Now, on the target machine we can install that TinyJail:

```console
# appjail jail create -I tiny+import=hello.appjail hello_tiny
...
# appjail start hello_tiny
...
# appjail cmd jexec hello_tiny hello
Hello, world!
```

We can use `pkg-query(8)` to export a package and its files in a TinyJail. To automate getting the dependencies, we use the following script:

```sh
#!/bin/sh

main()
{
	local jail="$1" package="$2"

	if [ -z "${jail}" -o -z "${package}" ]; then
		echo "usage: pkg.sh jail package" >&2
		exit 64
	fi

	get_depends "${jail}" "${package}" | while IFS= read -r _package
	do
		pkg -j "${jail}" query "%Fp" -- "${_package}"
	done
}

get_depends()
{
	_get_depends "$@" | sort | uniq
}

_get_depends()
{
	local jail="$1" package="$2"

	if [ -z "${jail}" -o -z "${package}" ]; then
		echo "usage: get_depends jail package" >&2
		exit 64
	fi

	pkg -j "${jail}" query '%dn-%dv' -- "${package}" | while IFS= read -r _package
	do
		get_depends "${jail}" "${_package}"
	done

	printf "%s\n" "${package}"
}

main "$@"
```

To export the `nginx` jail, simply create it, install `www/nginx` and run the script to save its output to a file.

```sh
appjail quick nginx virtualnet="web:nginx default" nat expose=80 start overwrite
appjail pkg jail nginx install -y nginx
./pkg.sh nginx nginx > nginx.lst
```

Before we can export the jail, we need to edit the list to add the `/usr/local/www` and `/usr/local/etc/nginx` directories and not just add the `*-dist` files.

```sh
appjail stop nginx
appjail jail create -I tiny+export="files:nginx.lst output:nginx.appjail compress:xz" nginx
```

The template is not included, so we can use the `bridge.conf` template and edit its required parameters.

```sh
appjail jail create -t /usr/local/share/examples/appjail/templates/bridge.conf -I tiny+import=nginx.appjail tinynginx
appjail-config set -j tinynginx '${iface}=nginx'
appjail-config set -j tinynginx '${ext_iface}=em0'
appjail-config set -j tinynginx devfs_ruleset=10
appjail start tinynginx
```

Now, a bit of post-processing:

```console
# appjail sysrc jail tinynginx ifconfig_sb_nginx="SYNCDHCP"
...
# appjail cmd jexec tinynginx service netif start
...
# appjail cmd jexec tinynginx mkdir -p /var/log/nginx
# appjail cmd jexec tinynginx ln -s /usr/local/www/nginx-dist /usr/local/www/nginx
# appjail cmd jexec tinynginx service nginx oneenable
...
# appjail cmd jexec tinynginx service nginx start
...
# fetch -qo - http://192.168.1.104
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

---

**See also**:

* [Virtual Networks](networking/virtual-networks/intro.md)
* [NAT](networking/virtual-networks/NAT.md)
* [Port Forwarding](networking/virtual-networks/port-forwarding.md)
