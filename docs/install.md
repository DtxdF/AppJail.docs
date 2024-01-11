If you want to contribute to the AppJail project or just want to get the latest features, use the bleeding-edge version:

```sh
git clone https://github.com/DtxdF/AppJail.git
cd AppJail
make install APPJAIL_VERSION=`make -V APPJAIL_VERSION`+`git rev-parse HEAD`
```

Another way to get the latest AppJail features is to install `sysutils/appjail-devel` port:

```sh
pkg install -y appjail-devel
```

Or if you prefer a much more stable version:

```sh
pkg install -y appjail
```

AppJail installed from your package manager or from the ports framework will give you an AppJail configuration file with useful comments, usually `/usr/local/etc/appjail/appjail.conf`. If you are installing AppJail from source code, copy `/usr/local/share/examples/appjail/appjail.conf` to `/usr/local/etc/appjail/appjail.conf`.

## Services/RC Scripts

AppJail comes with some useful RC scripts (or services, in some contexts). All of them are optional, but can provide you with useful features.

### etc/rc.d/appjail

This RC script is responsible for starting the jails at system startup. If you want to start your jails at startup, enable this service.

```sh
sysrc appjail_enable=YES
```

### etc/rc.d/appjail-dns

See [DNS](networking/DNS.md) for details.

### etc/rc.d/appjail-health

See [Supervisor/Healthcheckers](healthcheckers.md) for details.

### etc/rc.d/appjail-natnet

See [NAT](networking/virtual-networks/NAT.md) for details.
