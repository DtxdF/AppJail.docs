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

**See also**:

* [Virtual Networks](intro.md)
* [NAT](NAT.md)
