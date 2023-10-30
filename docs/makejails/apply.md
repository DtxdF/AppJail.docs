Maybe you have a jail already created to which you want to make some changes, but simply running a Makejail is not useful for your case since it will be recreated, so you create a script in the language of your choice, but you realize that you need to write more things than simply creating a Makejail. The solution to this problem is to apply a Makejail to an existing jail to take advantage of the Makejail instructions.

Suppose we create a jail that has `xrdp` and some DE like `lxde` and we forget to install some applications that we use daily, so to solve it we create a Makejail that by convention we name `Makejail.apply`.

**Makejail.apply**:

```
STAGE apply

PKG telegram-desktop \
    xpdf \
    librewolf
    mesa-dri
```

As you can see, we need to put `STAGE apply` before the instructions are executed.

To apply this Makejail just execute `appjail apply`.

```sh
appjail apply xrdp Makejail.apply
```

`appjail apply` does not require the jail to be started, it only needs an existing jail. This implies that some instructions that are intended to be executed in a started jail should not work.

## Notes

Note that applying a Makejail is not executed in the same way as executing instructions in the `build` stage. The `build` stage uses a buildscript (simply a script that is executed in this stage) and the `apply` stage uses an initscript that is temporarily generated to execute the instructions you defined in your Makejail. The problem is the actual directory as you can see below:

```console
# cat Makejail
OPTION start
OPTION overwrite=force

INCLUDE a/Makejail

CMD --local echo "main:"
CMD --local pwd
# cat a/Makejail
INCLUDE b/Makejail

CMD --local echo "a:"
CMD --local pwd
# cat a/b/Makejail
INCLUDE c/Makejail

CMD --local echo "b:"
CMD --local pwd
# cat a/b/c/Makejail
CMD --local echo "c:"
CMD --local pwd
# appjail makejail -j jtest
...
c:
/tmp/a/a/b/c
b:
/tmp/a/a/b
a:
/tmp/a/a
main:
/tmp/a
...
```

As you can see above, by not defining any stage the `build` stage is used in those Makejails, so the current directory is implicitly defined. But if we use the `apply` stage:

```console
# cat Makejail
OPTION start
OPTION overwrite=force

INCLUDE a/Makejail

STAGE apply

CMD --local echo "main:"
CMD --local pwd
# cat a/Makejail
INCLUDE b/Makejail

STAGE apply

CMD --local echo "a:"
CMD --local pwd
# cat a/b/Makejail
INCLUDE c/Makejail

STAGE apply

CMD --local echo "b:"
CMD --local pwd
# cat a/b/c/Makejail
STAGE apply

CMD --local echo "c:"
CMD --local pwd
# appjail apply jtest
...
c:
/tmp/a
b:
/tmp/a
a:
/tmp/a
main:
/tmp/a
...
```
