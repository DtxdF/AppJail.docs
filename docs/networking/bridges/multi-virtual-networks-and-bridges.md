A jail can be in several virtual networks at the same time. This can be very useful to organize the jails in a logic that suits your environment.

For example, we can create three jails: `jdb`, a database server, `jweb`, a web server and `jdev`, a jail to develop our application that uses the services provided by other jails.

We will create three networks: `web` for web servers (e.g.: nginx, apache, php-fpm, APIs, etc.), `db` for databases (e.g.: `mongodb`, `mariadb`, `mysql`, `redis`, etc.) and `development`, jails that we will use only for development. Our jail for development must be in the networks where the other jails are.

Virtual networks cannot provide connections to the outside, so we cannot install any packages or upgrade them. We cant use NAT, which will be shown later, or bridges, which is the case in the current section.

```sh
#
# Networks
#

# db
appjail network add db 10.42.0.0/24
# web
appjail network add web 10.0.0.0/24
# development
appjail network add development 192.168.0.0/10

#
# Jails
#

# jdb
appjail quick jdb \
    virtualnet="db:jdb" \
    bridge="jdb iface:jext" \
    start
# jweb
appjail quick jweb \
    virtualnet="web:jweb" \
    bridge="jweb iface:jext" \
    start
# jdev
appjail quick jdev \
    virtualnet="db:jd1" \
    virtualnet="development:jd2" \
    virtualnet="web:jd3" \
    bridge="jdev iface:jext" \
    start
```

Now, we can configure the interfaces that are attached to the shared bridge to provide connections to the outside. The above example does not use DHCP or SLAAC, but you can use it.

After configuring the network parameters for our interfaces inside the jail, we can install software, configure them and when we are done, we probably don't want to get connection to the outside for the `jdb` and `jweb` jails.

```sh
appjail network detach jdb
appjail network detach jweb
```

!!! warning

    Remember to edit the respective templates so that the above changes persist, as described in `Bridges`.

---

**See also**:

* [Bridges](intro.md)
* [Virtual Networks](../virtual-networks/intro.md)
* [DHCP & SLAAC](../DHCP-and-SLAAC.md)
* [Templates](../../templates.md)
