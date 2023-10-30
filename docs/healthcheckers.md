Simply starting the jail is fine for almost all users, but what happens when the jail contains a service that should not be stopped? Software has bugs and at any time your favorite web server or the DBMS can crash.

Healthcheckers is an AppJail feature that can monitor your jail, your services or whatever you need.

```console
# Using appjail quick:
appjail quick jtest \
    virtualnet="development:jtest default" \
    nat \
    start \
    overwrite \
    healthcheck
# Manually:
appjail healthcheck set jtest
```

To list all healthcheckers of the given jail use `appjail healthcheck list`.

```console
# appjail healthcheck list jtest
NRO  ENABLED  NAME  STATUS  HEALTH_CMD            RECOVER_CMD
0    1        -     -       appjail status -q %j  appjail restart %j
```

This jail has one healthchecker. This healthchecker will run `appjail status -q %j` and if it fails it will run `appjail restart %j`. `%j` is replaced with the jail name, use `%` twice to escape it.

The above explanation is the idea on which the `healthcheckers` are based, but they are more complex. The detailed step-by-step is as follows:

1. Set the status to `starting`.
2. If the *start period* is greater than 0, the process sleeps for the indicated seconds.
3. Sleep the process for the given *interval*.
4. Execute the *health command*. If the *health type* is `host`, it executes the given command on the host, otherwise if it is `jail` it executes the command on the jail.
5. If the *timeout* (in seconds) is reached, the signal configured for the *health command* is sent.
6. The `SIGKILL` signal is sent to the *health command* when its *kill after* (in seconds) is reached. You should probably set it to be greater than its *timeout*.
7. If the *health command* is successful, set the status to `healthy` and repeat step 3, otherwise set the status to `failing` and if the current retry count is reached, continue with step 8, otherwise continue with step 3.
8. If the current total of recoveries is reached, set the status to `unhealthy` and close the healthchecker, otherwise add one to the recovery count and continue with step 9.
9. Execute the *recover command*. If the *recover type* is `host`, it executes the given command on the host, otherwise if it is `jail` it executes the command on the jail.
10. If the *timeout* (in seconds) is reached, the signal configured for the *recover command* is sent.
11. The `SIGKILL` signal is sent to the *recover command* when its *kill after* (in seconds) is reached. You should probably set it to be greater than its *timeout*.
12. If the *recover command* fails, set the status to `unhealthy` and close the healthchecker, otherwise set the status to `healthy` and continue with step 3.

All healthcheckers are run in parallel following the above steps.

As you can see, healthcheckers do not guarantee that a failing software will work forever, as it is not worth wasting resources on such software. You can see such limits that the healthchecker uses.

```console
# appjail healthcheck list jtest nro interval recover_kill_after recover_timeout recover_timeout_signal recover_total retries start_period timeout timeout_signal
NRO  INTERVAL  RECOVER_KILL_AFTER  RECOVER_TIMEOUT  RECOVER_TIMEOUT_SIGNAL  RECOVER_TOTAL  RETRIES  START_PERIOD  TIMEOUT  TIMEOUT_SIGNAL
0    30        180                 120              sigterm                 3              3        0             120      sigterm
```

To run the healthcheckers in the foreground we can use `appjail healthcheck run`.

```console
# appjail healthcheck run jtest
[00:00:00] [ debug ] [jtest] Starting healthchecking ...
[00:00:01] [ debug ] [jtest] Starting healthchecker (nro = 0) ...
[00:00:01] [ debug ] [jtest] All healthcheckers has been started. Waiting...
[00:00:02] [ debug ] [jtest] Healthchecker (nro = 0, retries = 3, interval = 30, type = host, cmd = appjail status -q "jtest", timeout = 120, timeout_signal = sigterm, kill_after = 180)
[00:00:02] [ debug ] [jtest] Recover (nro = 0, total = 3, type = host, cmd = appjail restart "jtest", timeout = 120, timeout_signal = sigterm, kill_after = 180)
[00:00:02] [ debug ] [jtest] Sleeping (nro = 0, context = health, type = host, cmd = appjail status -q "jtest", interval = 30) ...
[00:00:32] [ debug ] [jtest] Executing (nro = 0, context = health, type = host, cmd = appjail status -q "jtest") ...
[00:00:33] [ debug ] [jtest] Sleeping (nro = 0, context = health, type = host, cmd = appjail status -q "jtest", interval = 30) ...
```

For demostration purposes, we can stop in another console the jail and see what happens.

```console
...
[00:04:08] [ debug ] [jtest] Failing (nro = 0, context = health, type = host, cmd = appjail status -q "jtest", status = 1, retry = 1:3)
[00:04:08] [ debug ] [jtest] Sleeping (nro = 0, context = health, type = host, cmd = appjail status -q "jtest", interval = 30) ...
```

And if we list the healthcheckers and look at the status we get the following:

```console
# appjail healthcheck list jtest
NRO  ENABLED  NAME  STATUS   HEALTH_CMD            RECOVER_CMD
0    1        -     failing  appjail status -q %j  appjail restart %j
```

After a while, our healthchecker will do its job.

```console
...
[00:04:08] [ debug ] [jtest] Failing (nro = 0, context = health, type = host, cmd = appjail status -q "jtest", status = 1, retry = 1:3)
[00:04:08] [ debug ] [jtest] Sleeping (nro = 0, context = health, type = host, cmd = appjail status -q "jtest", interval = 30) ...
[00:04:38] [ debug ] [jtest] Executing (nro = 0, context = health, type = host, cmd = appjail status -q "jtest") ...
[00:04:39] [ debug ] [jtest] Failing (nro = 0, context = health, type = host, cmd = appjail status -q "jtest", status = 1, retry = 2:3)
[00:04:39] [ debug ] [jtest] Sleeping (nro = 0, context = health, type = host, cmd = appjail status -q "jtest", interval = 30) ...
[00:05:09] [ debug ] [jtest] Executing (nro = 0, context = health, type = host, cmd = appjail status -q "jtest") ...
[00:05:10] [ debug ] [jtest] Failing (nro = 0, context = health, type = host, cmd = appjail status -q "jtest", status = 1, retry = 3:3)
[00:05:10] [ debug ] [jtest] Executing (nro = 0, context = recover, type = host, cmd = appjail restart "jtest", total = 0:3) ...
[00:05:11] [ warn  ] [jtest] Restarting jtest ...
[00:05:11] [ warn  ] [jtest] jtest is not running.
[00:05:12] [ debug ] [jtest] Locking jtest ...
[00:05:12] [ info  ] [jtest] Starting jtest...
...
```

Check the status again:

```console
# appjail healthcheck list jtest
NRO  ENABLED  NAME  STATUS   HEALTH_CMD            RECOVER_CMD
0    1        -     healthy  appjail status -q %j  appjail restart %j
```

A better way to run healthcheckers is to use the RC script.

```console
# sysrc appjail_health_enable=YES
appjail_health_enable:  -> YES
# service appjail-health start
AppJail log file (Health): /var/log/appjail.log
Starting appjail_health.
# appjail logs | grep -F '2023-05-29.log' | grep 'jtest' | grep 'healthcheckers'
jails     jtest                               healthcheckers  2023-05-29.log
# appjail logs tail jails/jtest/healthcheckers/2023-05-29.log -f
[00:00:00] [ debug ] [jtest] Starting healthchecking ...
[00:00:01] [ debug ] [jtest] Starting healthchecker (nro = 0) ...
[00:00:01] [ debug ] [jtest] All healthcheckers has been started. Waiting...
[00:00:02] [ debug ] [jtest] Healthchecker (nro = 0, retries = 3, interval = 30, type = host, cmd = appjail status -q "jtest", timeout = 120, timeout_signal = sigterm, kill_after = 180)
[00:00:02] [ debug ] [jtest] Recover (nro = 0, total = 3, type = host, cmd = appjail restart "jtest", timeout = 120, timeout_signal = sigterm, kill_after = 180)
[00:00:02] [ debug ] [jtest] Sleeping (nro = 0, context = health, type = host, cmd = appjail status -q "jtest", interval = 30) ...
[00:00:32] [ debug ] [jtest] Executing (nro = 0, context = health, type = host, cmd = appjail status -q "jtest") ...
[00:00:32] [ debug ] [jtest] Sleeping (nro = 0, context = health, type = host, cmd = appjail status -q "jtest", interval = 30) ...
[00:01:02] [ debug ] [jtest] Executing (nro = 0, context = health, type = host, cmd = appjail status -q "jtest") ...
[00:01:03] [ debug ] [jtest] Sleeping (nro = 0, context = health, type = host, cmd = appjail status -q "jtest", interval = 30) ...
```

We are not limited to using a single healthchecker. For example, if we want to monitor not only the jail but also a service, say, NGINX.

```console
# appjail pkg jail jtest install -y nginx
...
# appjail service jail jtest nginx oneenable
nginx enabled in /etc/rc.conf
# appjail service jail jtest nginx start
Performing sanity check on nginx configuration:
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful
Starting nginx.
# appjail healthcheck list jtest
NRO  ENABLED  NAME  STATUS   HEALTH_CMD            RECOVER_CMD
0    1        -     healthy  appjail status -q %j  appjail restart %j
# appjail healthcheck set -h 'jail:service nginx status' -r 'jail:service nginx start' jtest
# appjail healthcheck list jtest
NRO  ENABLED  NAME  STATUS   HEALTH_CMD            RECOVER_CMD
0    1        -     healthy  appjail status -q %j  appjail restart %j
1    1        -     -        service nginx status  service nginx start
# service appjail-health restart
Stopping appjail_health.
Waiting for PIDS: 65861.
AppJail log file (Health): /var/log/appjail.log
Starting appjail_health.
# tail /var/log/appjail.log
[00:00:00] [ debug ] Starting jtest's healthcheckers; Log jails/jtest/healthcheckers/2023-05-29.log;
[00:00:01] [ debug ] Wait...
# appjail logs tail jails/jtest/healthcheckers/2023-05-29.log
[00:00:03] [ debug ] [jtest] Sleeping (nro = 1, context = health, type = jail, cmd = service nginx status, interval = 30) ...
[00:00:33] [ debug ] [jtest] Executing (nro = 0, context = health, type = host, cmd = appjail status -q "jtest") ...
[00:00:33] [ debug ] [jtest] Executing (nro = 1, context = health, type = jail, cmd = service nginx status) ...
nginx is running as pid 48490.
[00:00:33] [ debug ] [jtest] Sleeping (nro = 1, context = health, type = jail, cmd = service nginx status, interval = 30) ...
[00:00:33] [ debug ] [jtest] Sleeping (nro = 0, context = health, type = host, cmd = appjail status -q "jtest", interval = 30) ...
[00:01:03] [ debug ] [jtest] Executing (nro = 1, context = health, type = jail, cmd = service nginx status) ...
[00:01:03] [ debug ] [jtest] Executing (nro = 0, context = health, type = host, cmd = appjail status -q "jtest") ...
nginx is running as pid 48490.
[00:01:04] [ debug ] [jtest] Sleeping (nro = 1, context = health, type = jail, cmd = service nginx status, interval = 30) ...
```

---

**See also**:

* [Virtual Networks](networking/virtual-networks/intro.md)
* [NAT](networking/virtual-networks/NAT.md)
* [Logs](logs.md)
