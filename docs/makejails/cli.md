At first it may seem that to run a Makejail we need another Makejail that includes it. For a long term application this is useful, as we need to remember all the options to rebuild another jail in the near future, but from time to time we need to pass options from the command-line.

```sh
appjail makejail \
    -j nscde \
    -o virtualnet="development:nscde default" \
    -o nat \
    -o copydir=/tmp/files \
    -o file=/etc/rc.conf \
    -o x11 \
    -f gh+AppJail-makejails/nscde
```

Using a single command the above command creates a jail named `nscde` to run the `nscde` desktop environment.
