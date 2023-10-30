As we have mentioned `appjail image metadata info` is the command to get information about an image, so with it we can know if we should update them or we can even know which images are installed.

```console
# appjail image metadata info
Name            :    gonic (latest)
Build on        :
  - amd64
Image           :    amd64
  - SHA256 = 342ffef8f02f792d1b98f56972bd6632dd71a21ee271bebb50d20cd788eb2bde
  - SIZE = 117669148
  - TIMESTAMP = Wed Jun 21 16:37:31 2023
Source          :    amd64
  - https://dtxdf.duckdns.org/j1ucU8x3z6/gonic_latest_amd64_image_appjail
  - https://anonget.onrender.com/j1ucU8x3z6/gonic_latest_amd64_image_appjail
Name            :    gonic (minimal)
Build on        :
  - amd64
Image           :    amd64
  - SHA256 = b3179c173ea9cf0170303ad0b3efc443038edc8f3b8a91b8ec6f2f80603abae3
  - SIZE = 12404300
  - TIMESTAMP = Wed Jun 21 17:21:34 2023
Source          :    amd64
  - https://dtxdf.duckdns.org/N9t6U0x1z5/gonic_minimal_amd64_image_appjail
  - https://anonget.onrender.com/N9t6U0x1z5/gonic_minimal_amd64_image_appjail
Installed       :
  - /usr/local/appjail/cache/images/gonic/minimal-amd64-image.appjail
Entrypoint      :    gh+AppJail-makejails/gonic
AJSPEC          :    .ajspec
Name            :    hello (latest)
Build on        :
  - amd64
Image           :    amd64
  - SHA256 = c01698b9f15204ec243153fda2df4cff11c348349c550105b7b764b0f4a0130d
  - SIZE = 457976
  - TIMESTAMP = Thu Jun 15 16:06:25 2023
Source          :    amd64
  - https://dl.dropboxusercontent.com/s/fac2ujzw1idl8xg/latest-amd64-image.appjail?dl=0
Installed       :
  - /usr/local/appjail/cache/images/hello/latest-amd64-image.appjail
Entrypoint      :    gh+AppJail-makejails/hello
AJSPEC          :    .ajspec
```

The `Installed` keyword is very important because it specifies which images are installed. As you can see, only the `gonic` image with the `latest` tag is missing, so it is not installed.

If we no longer want an image, we can simply remove it using `appjail image remove`.

```console
# appjail image remove hello
# appjail image list
NAME
gonic
```

**WARNING**: When using ZFS as the backend file system `appjail image remove` will recursively remove all datasets including all references, such as clones. Be careful.

But what happens when we want to remove a specific tag?

```console
# appjail image remove -t minimal gonic
/usr/local/appjail/cache/images/gonic/.done-minimal-amd64-image.appjail
/usr/local/appjail/cache/images/gonic/minimal-amd64-image.appjail
```

When we pass `-t` to `appjail image remove` AppJail will remove that specific tag but for all architectures. We can specify the architecture and the tag.

```console
# appjail image remove -a amd64 -t latest hello
/usr/local/appjail/cache/images/hello/.done-latest-amd64-image.appjail
/usr/local/appjail/cache/images/hello/latest-amd64-image.appjail
```

The same can happen when we need to remove all tags but for a specific architecture.

```console
# appjail image remove -a amd64 gonic
/usr/local/appjail/cache/images/gonic/.done-latest-amd64-image.appjail
/usr/local/appjail/cache/images/gonic/latest-amd64-image.appjail
/usr/local/appjail/cache/images/gonic/.done-minimal-amd64-image.appjail
/usr/local/appjail/cache/images/gonic/minimal-amd64-image.appjail
```

---

**See also**:

* [Images](intro.md)
* [AJSPEC](ajspec.md)
