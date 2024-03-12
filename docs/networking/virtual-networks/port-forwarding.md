A jail can act as a service that our clients can take advantage of, but if we use NAT instead of bridging, we need to forward a host port to a jail port. Fortunately, AppJail can do that in a very easy way.

!!! warning

    You must read [Packet Filter](../packet-filter.md) before using this option.

For example, NGINX can be used in a jail to provide a web server to our customers:

```sh
# Using appjail quick:
appjail quick nginx virtualnet="web:nginx default" nat expose=80 start
# Manually:
appjail quick nginx virtualnet="web:nginx default" nat overwrite
appjail expose set -k web -p 80 nginx
appjail-config set -Ij nginx exec.prestart='appjail expose on ${name}'
appjail-config set -Ij nginx exec.prestart='appjail expose off ${name}'
appjail start nginx
# www/nginx:
appjail pkg jail nginx install -y nginx
appjail sysrc jail nginx nginx_enable="YES"
appjail service jail nginx nginx start
```

Since we are the host we can make an HTTP request using the jail's IPv4 address (assuming it is `10.0.0.2`).

```console
# fetch -qo - http://10.0.0.2
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

Another host on the host network can use the host's IPv4 address (assuming it is `192.168.1.105`) to make an HTTP request as in the example above.

```console
# fetch -qo - http://192.168.1.105
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

You may encounter in a situation where an application requires you open a range of ports to work properly. AppJail can expose a range of ports.

```console
# appjail quick netcat \
    overwrite=force \
    start \
    expose="8000-9000:8000" \
    virtualnet=":<random> default" \
    nat
...
# appjail cmd jexec netcat \
    nc -v -l 8000
Connection from 192.168.2.106 40485 received!
```

In the above case, an external client can connect to our service and the firewall performs a redirect to the jail's IP address and any valid port in the range we specify. Once the redirection is done, port 8000 is used as the destination port, but the external client does not perceive this, only the application.

```console
# nc -z -v 192.168.2.101 8500
Connection to 192.168.2.101 8500 port [tcp/*] succeeded!
```

The expose option supports a variety of syntaxes to achieve a variety of use cases:

* `port`: An external client can connect from the given port to an application listening on the same port.
* `hport:jport`: An external client can connect from the given port (`hport`) to an application listening on the same or a different port (`jport`).
* `min-max`: An external client can connect from the given range of ports to the same range of ports. For example, if the external client connects to port `8000`, the application should listen on port `8000`, if the external client connects to port `8001`, the application should listen on port `8001`, etc.
* `min-max:jport`: An external client can connect from a given range of ports to an application listening on the given port. See the Netcat example above.
* `hport:min-max`: An external client can connect to the given port to a range of ports. AppJail supports this syntax due to the flexibility of the expose option, but this use case is useless in many cases because the client can only connect to the port given to the first in the range of ports, so, for example, if you specify `8000:5555-5999`, the client connects to `8000` and the application should listen on `5555`; listening in another will not work.
* `min-max:min-max`: An external client can connect from the given range of ports to same or a different range of ports. See `min-max` for more details.
* `hport:-max`: This works in the same way as `hport:min-max`. `hport` is used as the missing part: `min`.

---

**See also**:

* [Virtual Networks](intro.md)
* [NAT](NAT.md)
