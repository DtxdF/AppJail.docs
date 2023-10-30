You can assign some specific CPUs to a jail for more resource constraint using the `cpuset` option in `appjail quick`:

```sh
appjail quick jtest cpuset="2-4,5,7" start
```
