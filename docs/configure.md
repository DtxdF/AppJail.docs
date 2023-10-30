As mentioned in [Getting Started](getting-started.md), AppJail is configured dynamically to make it much easier to use, however, the payoff is performance and reliability. Performance degrades because AppJail needs to execute commands to get information from your system and reliability degrades because those values are obtained dynamically and can change at any time depending on some factors. Fortunately, configuring AppJail is very easy.

!!! note

    AppJail configuration file has many options, we just need to set some useful options
    for better usability, performance and reliability.

## EXT\_IF

External Interface. In almost all cases, the interface you use to access the network.

## ON\_IF

The name or group of the network interface to transmit packets on. In almost all cases, it must have the same value as `EXT_IF`.

## FREEBSD\_VERSION

FreeBSD version of the current system.

## FREEBSD\_ARCH

FreeBSD architecture of the current system.

## IMAGE\_ARCH

Default architecture used by AppJail images.

**See also**:

* [Images](images/intro.md)

## SHORTEN\_DOMAIN\_NAMES

It is used to shorten the domain name of your jails, so that you can communicate between them using only their name, such as `redis` instead of `redis.ajnet.appjail` when using the DNS system.

**See also**:

* [DNS#using-shorter-domain-names](networking/DNS.md#using-shorter-domain-names)

## ENABLE\_ZFS

If you plan to take advantage of ZFS with AppJail, set this option.

!!! warning

    Do not mix installations that already have this option disabled, change `PREFIX`
    or re-import jails.

**See also**:

* [ZFS](ZFS.md)

## Configuration file (example)

**/usr/local/etc/appjail/appjail.conf**:

```
EXT_IF=jext
ON_IF=jext
FREEBSD_VERSION=13.2-RELEASE
FREEBSD_ARCH=amd64
IMAGE_ARCH=amd64
SHORTEN_DOMAIN_NAMES=1
# Remove the # character if you want to use ZFS with AppJail.
#ENABLE_ZFS=1
```
