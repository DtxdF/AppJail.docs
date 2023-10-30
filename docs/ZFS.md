AppJails supports ZFS to take advantage of its benefits, such as snapshotting, cloning, fast backups, quotas, and so on. To enable ZFS in AppJail, simply change `ENABLE_ZFS` to `1` in your AppJail configuration file.

!!! warning

    Do not mix UFS and ZFS installations, change `PREFIX` and other ZFS related options
    and migrate the jails by importing and exporting them.

There are other ZFS related options that you can configure for your environment. `ZPOOL` (default: `zroot`), the pool name. `ZROOTFS` (default: `appjail`) will be concatenated with `ZPOOL` to be used as a prefix for datasets. `ZOPTS` (default: `-o compress=lz4`), options for `zfs-create(8)`.

---

**See also**:

* [Backups](backups/intro.md)
