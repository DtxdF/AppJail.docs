To take advantage of the images we can use an image in a Makejail using the `FROM` instruction. After the instruction takes effect, the new jail will have the files contained in the image.

```
FROM --entrypoint gh+DtxdF/webapp-image webapp:py

OPTION start
OPTION overwrite
OPTION virtualnet=:webapp01 default
OPTION nat
```

All we have to do is run `appjail-makejail(1)`.

```sh
appjail makejail -j webapp01
```

---

**See also**:

* [Images](intro.md)
