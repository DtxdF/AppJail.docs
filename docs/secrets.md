In AppJail, a secret is any blob of data, such as passwords, SSH private keys, TLS keys, or data that should not be publicly accessible, except from authorized jails, in read-only mode, with restricted permissions, and stored encrypted on disk.

Of course, for jails to be able to use secrets at any given time, they must be available in unencrypted form. In AppJail, unencrypted secrets are stored in an in-memory filesystem that isn't swap-backed, so they are always available from memory and never touch permanent storage.

## How AppJail manages secrets

When you add a secret using the `appjail secrets create` command, it is encrypted and stored on disk. Currently, there are four ways to create a secret:

1. From standard input.
2. From a file.
3. From a string.
4. From another secret.

AppJail uses a backend specified by the `SECRETS_BACKEND` parameter in your `appjail.conf(5)` file to encrypt and decrypt data. Encryption and decryption should not be performed interactively, so any data such as the encryption key used by the backend is stored in plaintext. If this concerns you, you can change the `SECRETS_BACKENDSDIR` parameter to a partition or ZFS dataset with encryption enabled.

Secrets are not stored as a single entity but are organized into a group of secrets. For example, if you create a new secret named `postgres/passwd`, the `appjail secrets create` command will create the `postgres` group if it does not already exist, and the `passwd` secret. You can specify a custom UID, GID, and file mode if you wish, but in most cases, the secrets are used by initialization scripts that run as root, so the default UID and GID are `0`, and the file mode is `0400`. You can change this to an insecure file mode or to a different UID or GID, since `SECRETSDIR` (the working directory of `appjail-secrets(1)`) has a restricted file mode (`0700`).

To use an existing secret, `appjail-fstab(1)` is the tool that handles this, since the secret is added as an entry to the jail's `fstab(5)` file. However, this entry sets the filesystem type to `<secretsfs>`, a pseudo-file system. This filesystem type instructs `appjail-fstab(1)` to call `appjail secrets use`, which creates the memory-backed device with the in-memory filesystem where the unencrypted secrets will be stored. Once this is done, this pseudo-filesystem works in the same way as `nullfs`, but resolves the secret group into a valid path from the host that points to the in-memory file system; however, this is handled internally, so you don’t even need to worry about it.

The recommended way to use a secret is through the `secret` option in `appjail-quick(1)`. This is because this option invokes `appjail secrets use` not only to create the memory-backed device and the in-memory filesystem, but also to check for the existence of the the group before adding an entry to the jail’s `fstab(5)`. This option also correctly sets `options` to `ro` to mount the secret group in read-only mode, which is highly recommended and should not be modified in any way unless you know what you are doing.

The `secret` option in `appjail-quick(1)` also sets the `mountpoint` to the expected location: `/secrets/<group>`. Jails can access secrets just like any other file in its directory. For example, the previous case `postgres/passwd` will create a mountpoint at `/secrets/postgres` and the secret will be available as `/secrets/postgres/passwd`.

## Examples

**Creating a new secret from a string**:

```sh
appjail secrets create -s mysecrets/passwd 123
```

**Creating a new secret from standard input**:

```sh
echo 123 | appjail secrets create mysecrets/passwd
```

**Creating a new secret from a file**:

```sh
appjail secrets create webserver/server.crt server.crt
```

**List existing secret groups**:

```console
$ appjail secrets ls
mysecrets
webserver
```

Using the `-R` option tells `ls` to use the in-memory file system instead.

```console
$ appjail secrets ls -R
webserver
```

In the previous example, no one is using `mysecrets`, so it hasn't been created in the in-memory file system.

**List all the secrets available in a secret group**:

```console
$ appjail secrets ls mysecrets
passwd
```

**Decrypt a secret**:

```console
$ appjail secrets cat mysecrets/passwd
123
```

**Remove a secret**:

```console
$ appjail secrets rm mysecrets/passwd
$ appjail secrets ls
webserver
```

If the secret group no longer contains any secrets, it is considered nonexistent.

**Remove all secrets from a group**:

```sh
appjail secrets rmdir webserver
```

**Update a secret**:

```console
$ appjail secrets create -s mysecrets/passwd 123
$ appjail secrets update -s mysecrets/passwd 321
$ appjail secrets cat mysecrets/passwd
321
```

When you update or remove a secret, in addition to updating or removing it from disk, it is also updated or removed from the in-memory filesystem if the secret exists there. Regarding updates, if you do not specify a UID, GID, or file mode, secrets inherit those attributes from the encrypted secret.

**Creating a jail with secrets**:

```console
$ appjail quick jsecret overwrite=force start secret=mysecrets ephemeral
...
[00:00:01] [ debug ] [jsecret] Configuring secrets ...
[00:00:01] [ debug ] [jsecret] Allowing secret 'mysecrets'
...
$ appjail fstab jail jsecret
NRO  ENABLED  NAME  DEVICE     MOUNTPOINT          TYPE         OPTIONS  DUMP  PASS
0    1        -     mysecrets  /secrets/mysecrets  <secretsfs>  ro       0     0
$ appjail cmd jexec jsecret ls -lR /secrets
total 4
drwxr-xr-x  2 root wheel 512 20 ago.  16:28 mysecrets

/secrets/mysecrets:
total 4
-r--------  1 root wheel 4 20 ago.  16:28 passwd
$ appjail cmd jexec jsecret rm -f /secrets/mysecrets/passwd
rm: /secrets/mysecrets/passwd: Read-only file system
$ appjail cmd jexec jsecret sh -c "echo 1 > /secrets/mysecrets/passwd"
sh: cannot create /secrets/mysecrets/passwd: Read-only file system
$ appjail cmd jexec jsecret cat /secrets/mysecrets/passwd
321
$ appjail secrets update -s mysecrets/passwd 1234
$ appjail cmd jexec jsecret cat /secrets/mysecrets/passwd
1234
```

### Memory-backed and the in-memory filesystem size

Keep in mind that a secret must be small enough to fit within the default size specified by the `SECRETS_MDMFS_SIZE` parameter, which defaults to 32 MiB. If this size does not meet your needs, increase it to a realistic value. If you want to store blob of data that are not small, consider using disk encryption.

---

**See also**:

* [File System Management](fs-mgmt.md)
