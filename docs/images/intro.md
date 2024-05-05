Analogously for a program, a Makejail is like a source code and the Image is like the resulting executable file. Technically they are independent, but when combined they are useful for some types of applications.

Suppose we are writing an application that depending on the installed dependencies will provide us with more or less features and we want to share all the variants with a co-worker. What we have to do is to create a jail, export it and put it in a place that our co-worker can download, but he doesn't need to manually download the image, he needs an [ajspec file](ajspec.md).

## Getting started with Images

To create an image we first need an existing jail, then we can make changes to custumize it, stop the jail and finally export it. A very important step that will probably be necessary after stopping the jail and before exporting it is to remove sensitive data such as private keys and/or non-portable files like `/etc/rc.conf`. Of course, removing `/etc/rc.conf` is unnecessary when we know the non-portable parameters that can affect the jail when it is installed on the target (e.g.: `defaultrouter`).

```console
# appjail quick webapp virtualnet=":webapp default" nat start overwrite
...
# appjail login webapp
...
# appjail stop webapp
...
# appjail image export -t py -c zstd webapp
[00:00:01] [ info  ] [webapp] Exporting webapp ...
[00:00:01] [ debug ] [webapp] Generating /usr/local/appjail/cache/images/webapp/py-amd64-image.appjail ...
[00:00:02] [ info  ] [webapp] Done.
[00:00:02] [ debug ] [webapp] Setting (ajspec): tags: py
[00:00:02] [ debug ] [webapp] Setting (ajspec): py.arch: amd64
[00:00:03] [ debug ] [webapp] Setting (ajspec): py.name: "webapp"
[00:00:03] [ debug ] [webapp] Setting (ajspec): py.timestamp: 1687814074
[00:00:03] [ debug ] [webapp] Setting (ajspec): py.sum.amd64: 7317ab3cb8bb03b48b1df7187b032dc7a99b5cb4b2a500eac88a0e1c4603ffc6
[00:00:03] [ debug ] [webapp] Setting (ajspec): py.size.amd64: 847852
[00:00:03] [ info  ] [webapp] Saved as /usr/local/appjail/cache/images/webapp/py-amd64-image.appjail
# appjail image metadata info webapp
Name            :    webapp (py)
Build on        :
  - amd64
Image           :    amd64
  - SHA256 = 7317ab3cb8bb03b48b1df7187b032dc7a99b5cb4b2a500eac88a0e1c4603ffc6
  - SIZE = 847852
  - TIMESTAMP = Mon Jun 26 17:14:34 2023
Installed       :
  - /usr/local/appjail/cache/images/webapp/py-amd64-image.appjail
```

After exporting a jail as an image we will probably want to make it public. You need to put the ajspec in a safe place that ensures it is only handled by trusted people and you can put the image anywhere, but it is advisable to put it in a safe place if you can. Remember that AppJail when importing an image will check the checksum to see if they match, so the ajspec file must be in a safe place.

In this case we will use Github to save our ajspec file in our repository. The image will be placed in two servers, because if the first one fails, AppJail will try with the second one, and if it fails with the third one, and so on.

```console
# scp \
    /usr/local/appjail/cache/images/webapp/py-amd64-image.appjail \
    root@192.168.1.107:/var/mks/makejails/darkhttpd/appdata/AppJail-images/webapp
...
# cp /usr/local/appjail/cache/images/webapp/py-amd64-image.appjail /tmp/imgs/AppJail-images/webapp
# git clone git@github.com:DtxdF/webapp-image
...
# cp /usr/local/appjail/cache/images/webapp/.ajspec webapp-image
# appjail image metadata set -t py -f webapp-image/.ajspec source:amd64="http://localhost:8080/AppJail-images/webapp/py-amd64-image.appjail"
# appjail image metadata set -t py -f webapp-image/.ajspec source:amd64+="http://192.168.1.107:8080/AppJail-images/webapp/py-amd64-image.appjail"
# git -C webapp-image add .ajspec
# git -C webapp-image commit -m 'Add ajspec file'
# git -C webapp-image push
...
```

Now anyone can import the image (assuming the image is placed on a public site).

```console
# appjail image import -t py gh+DtxdF/webapp-image
[00:00:01] [ debug ] Cloning https://github.com/DtxdF/webapp-image as /usr/local/appjail/cache/tmp/.appjail/appjail.UWiaAZq4 ...
[00:00:02] [ debug ] [webapp] Fetching webapp from http://localhost:8080/AppJail-images/webapp/py-amd64-image.appjail: fetch -Rpm -o "/usr/local/appjail/cache/images/webapp/py-amd64-image.appjail" "http://localhost:8080/AppJail-images/webapp/py-amd64-image.appjail"
/usr/local/appjail/cache/images/webapp/py-amd6         828 kB   95 MBps    00s
[00:00:02] [ info  ] [webapp] Saved as /usr/local/appjail/cache/images/webapp/py-amd64-image.appjail
# appjail image list
NAME
webapp
# appjail image jail -t py -i webapp webapp01 virtualnet=":webapp01 default" nat start overwrite
...
```

!!! note

    When retrieving an ajspec file on import, it will overwrite the current ajspec file.

AppJail can update images for you. You just need to use `appjail-image(1)` `update`.

```console
# appjail image update
[00:00:00] [ info  ] [webapp] Updating webapp (arch:amd64, tag:py) ...
[00:00:00] [ debug ] [webapp] Cloning https://github.com/DtxdF/webapp-image as /usr/local/appjail/cache/tmp/.appjail/appjail.ih3dse3c ...
[00:00:01] [ info  ] [webapp] webapp (arch:amd64, tag:py): already up to date.
```
