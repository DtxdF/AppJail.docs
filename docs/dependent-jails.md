AppJail can start jails before the jail we are specifying. It works in the same way as dependencies. For example, if we have the NGINX, MariaDB and PHP-FPM jails, we want to start MariaDB, PHP-FPM and NGINX in this order. The idea is to run `appjail start nginx` and AppJail will recursively start the `nginx` jail dependencies first.

We just have to put the `depend` parameter in the template and `appjail-start(1)` will do the job of starting dependencies.

```sh
appjail-config set -j nginx depend=php-fpm
appjail-config set -j php-fpm depend=mariadb
``` 

The `appjail-stop(1)` command will not stop the dependencies because the clients may be using the services offered by those jails. But, AppJail has a command to recursively stop the jail and its dependencies named `appjail-rstop(1)`.

`appjail-rstop(1)` sorts the jails in the same way as `appjail-start(1)` does but in reverse order.

```sh
appjail rstop nginx
```

NGINX, PHP-FPM and MariaDB jails will be stopped in this order.

---

**See also**:

* [Templates](templates.md)
