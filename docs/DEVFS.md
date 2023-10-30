AppJail has support for DEVFS ruleset management which means that you can manage your devices within jails in a straighforward and dynamic way.

```console
# appjail quick jtest \
    overwrite=force \
    start \
    device='include $devfsrules_hide_all' \
    device='include $devfsrules_unhide_basic' \
    device='include $devfsrules_unhide_login'
...
# appjail devfs list jtest
NRO  ENABLED  NAME  RULE
0    1        -     include $devfsrules_hide_all
1    1        -     include $devfsrules_unhide_basic
2    1        -     include $devfsrules_unhide_login
```

The operation is very simple: `appjail quick` uses `appjail devfs set` to set your DEVFS rules. Before the jail starts using `appjail start`, it checks for rules; if so, it will load them using `appjail devfs load` and set `devfs_ruleset`. `appjail start` gets `devfs_ruleset` in one of two ways: if you set `devfs_ruleset` in your template, it chooses it, but if you don't, it will use an algorithm described below to get an available ruleset.

## Assigning a ruleset

AppJail uses `appjail devfs ruleset assign` which selects an available ruleset using the following algorithms:

* `fsmn` (Find Smallest Missing Number): This algorithm will select an smallest unused number from a list of numbers. It takes into account the length of the list. If the list has a length of `0`, the unused number is `1`; if the length is `1`, so the list has one element, the algorithm checks if that element is `1`, if so, the unused number is 2, if not, is `1`. If this check is unsuccessful, the algorithm does an N/2 linear search to compare two numbers starting from `1` up to the length of the list. If nothing matches, the last element plus `1` is the unused number.
* `fnfs` (Find Number From Start): This algorithm selects a given number from a list of numbers. If this number is already in use, the number is incremented and the search continues. Once this search is finished, the resulting number is the unused. As `fsmn`, it takes the length of the list. If the list has a length of `0`, the resulting number is the same as the given number.

**Recommendation**: Use `fsmn` if you don't have problems with assigning lower numbers, probably because you do not edit `devfs.rules(5)` or at least not often. It is recommended to use this algorithm if you set your rulesets number to a high number, such as `1000`. Use `fnfs` is you want a more deterministic way of assigning the ruleset number.

To change the algorithm, set `DEVFS_ASSIGN_ALGO` (default: `fsmn`) in your `appjail.conf`. If you choose `fnfs` set `DEVFS_FNFS` (default: `1000`) to set the starting number.

## Applying rules

To apply rules dynamically, use `appjail devfs apply`.

```console
# appjail cmd jexec jtest ls /dev
fd      null    pts     random  stderr  stdin   stdout  urandom zero
# appjail devfs apply jtest path bpf unhide
# appjail cmd jexec jtest ls /dev
bpf     fd      null    pts     random  stderr  stdin   stdout  urandom zero
```

## DEVFS and Makejails

Since we can use the `device` option in `appjail quick`, we can also use it in a Makejail.

```
OPTION overwrite=force
OPTION start
OPTION device=include \$devfsrules_hide_all
OPTION device=include \$devfsrules_unhide_basic
OPTION device=include \$devfsrules_unhide_login
OPTION device=path fuse unhide
OPTION device=path zfs unhide
```

As we can see, we need to escape the dollar sign to process it correctly, otherwise it will be processed as an (undefined) variable.

The big advantage of this approach is that it is very easy to use since we reuse the `appjail quick` options. The problem is that since the `OPTION` instruction is processed before some instructions, such as `RAW`, we cannot use it conditionally. However, the solution is the `DEVICE` instruction, as you can see below.

```
ARG enable_bpf?

OPTION overwrite=force
OPTION start

DEVICE include 1
DEVICE include 2
DEVICE include 3
DEVICE path fuse unhide
DEVICE path zfs unhide
RAW if [ -n "${enable_bpf}" ]; then
    DEVICE path bpf unhide
RAW fi
```

The disadvantage of the `DEVICE` instruction is that since it uses `appjail devfs apply`, we cannot reference other rulesets as when using the `device` option or `appjail devfs set`. Although it is easy to simulate this behavior.

```
ARG enable_bpf?

OPTION overwrite=force
OPTION start

RAW devfsrules_hide_all=1
RAW devfsrules_unhide_basic=2
RAW devfsrules_unhide_login=3

DEVICE include $devfsrules_hide_all
DEVICE include $devfsrules_unhide_basic
DEVICE include $devfsrules_unhide_login
DEVICE path fuse unhide
DEVICE path zfs unhide
RAW if [ -n "${enable_bpf}" ]; then
    DEVICE path bpf unhide
RAW fi
```

Run this Makejail and you will see the magic...

```console
# appjail makejail -j jtest -- --enabled_bpf yes
...
# appjail cmd jexec jtest ls /dev
bpf     fd      fuse    null    pts     random  stderr  stdin   stdout  urandom zero
```
