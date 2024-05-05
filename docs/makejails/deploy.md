Using the host to install a bunch of applications with their dependencies, configuring them and doing difficult things to simply use that application is a lot of work in some cases.

A perfect example is when a user **just wants to use PHP on a web server**. The user must install PHP and a web server such as NGINX. NGINX needs to be configured to be used in conjunction with `php-fpm`. `php-fpm` also needs to be configured if the user wants to use it with unix sockets.

In addition, all these applications must coexist with other applications on the host, which can be problematic in some cases.

Instead of using the host to perform the above process, the user can use a Makejail with the necessary instructions to run the application in a jail.

**Makejail**:

```
INCLUDE options/options.makejail
INCLUDE options/network.makejail
INCLUDE php/Makejail
INCLUDE nginx/Makejail

CMD mkdir -p /usr/local/www/php
CMD --local mkdir -p /usr/local/www/php
CMD --local chown www:www /usr/local/www/php
MOUNT /usr/local/www/php /usr/local/www/php
```

**options/options.makejail**:

```
OPTION start
OPTION mount_devfs
OPTION resolv_conf
OPTION tzdata
OPTION overwrite
```

**options/network.makejail**:

```
OPTION virtualnet=web:${APPJAIL_JAILNAME} default
OPTION nat
```

**php/Makejail**:

```
PKG php80
SYSRC php_fpm_enable=YES

COPY --verbose usr

CMD ln -s /usr/local/etc/php.ini-production /usr/local/etc/php.ini

SERVICE php-fpm start
```

**php/usr/local/etc/php-fpm.d/www.conf**:

```
[www]
user = www
group = www
listen = /var/run/php-fpm.sock
listen.owner = www;
listen.group = www;
pm = dynamic
pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3
```

**nginx/Makejail**:

```
PKG nginx
SYSRC nginx_enable=YES

COPY --verbose usr

SERVICE nginx start
```

**nginx/usr/local/etc/nginx/nginx.conf**:

```
events {
        worker_connections 1024;
}

http {
        include       mime.types;
        default_type  application/octet-stream;

        upstream php {
                server unix:/var/run/php-fpm.sock;
        }

        server {
                listen 80;
                server_name $hostname "";
                root /usr/local/www/php;
                index index.php;

                client_max_body_size 8M;

                location / {
                        try_files $uri $uri/ =404;
                }

                location ~ \.php$ {
                        include fastcgi_params;
                        fastcgi_intercept_errors on;
                        fastcgi_pass php;
                        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                }
        }
}
```

The above files have the following structure.

```console
# tree -pug
[drwxr-x--- root     wheel   ]  .
├── [-rw-r----- root     wheel   ]  Makejail
├── [drwxr-x--- root     wheel   ]  nginx
│   ├── [-rw-r----- root     wheel   ]  Makejail
│   └── [drwxr-xr-x root     wheel   ]  usr
│       └── [drwxr-xr-x root     wheel   ]  local
│           └── [drwxr-xr-x root     wheel   ]  etc
│               └── [drwxr-xr-x root     wheel   ]  nginx
│                   └── [-rw-r--r-- root     wheel   ]  nginx.conf
├── [drwxr-x--- root     wheel   ]  options
│   ├── [-rw-r----- root     wheel   ]  network.makejail
│   └── [-rw-r----- root     wheel   ]  options.makejail
└── [drwxr-x--- root     wheel   ]  php
    ├── [-rw-r----- root     wheel   ]  Makejail
    └── [drwxr-xr-x root     wheel   ]  usr
        └── [drwxr-xr-x root     wheel   ]  local
            └── [drwxr-xr-x root     wheel   ]  etc
                └── [drwxr-xr-x root     wheel   ]  php-fpm.d
                    └── [-rw-r--r-- root     wheel   ]  www.conf

11 directories, 7 files
```

Now, we can use `appjail-makejail(1)` to get a web server with PHP.

```console
# appjail makejail -j php
...
# echo '<?php echo "Hello, world!\n" ?>' > /usr/local/www/php/index.php
# fetch -qo - http://10.0.0.3
Hello, world!
```
