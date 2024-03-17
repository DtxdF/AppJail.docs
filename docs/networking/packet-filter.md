!!! tip
    Not all network options have packet filtering as a requirement, so you can skip
    this section if you are unsure. This section will be mentioned when necessary.

## PF

AppJail uses anchors like other applications that use `pf(4)` as a backend. Just enable `pf(4)` in `rc.conf(5)`, put the anchors in the `pf.conf(5)` file and reload the rules.

```sh
# Enable pf(4):
sysrc pf_enable="YES"
sysrc pflog_enable="YES"
# Put the anchors in pf.conf(5):
cat << "EOF" >> /etc/pf.conf
nat-anchor "appjail-nat/jail/*"
nat-anchor "appjail-nat/network/*"
rdr-anchor "appjail-rdr/*"
EOF
# Reload the pf(4) rules:
service pf reload
# Or reboot if you don't have pf(4) started:
service pf restart
service pflog restart
```

## Host configuration

Some network options need to forward packets (IPv4) between interfaces, so it is necessary to change some options.

```sh
# sysrc
sysrc gateway_enable="YES"
# sysctl
sysctl net.inet.ip.forwarding=1
```

## Default interfaces

Some network options require `EXT_IF` and `ON_IF` to be set in your AppJail configuration file. AppJail uses `EXT_IF` as the external interface that is normally used to obtain its IP address. The firewall uses `ON_IF` to operate on this interface.

AppJail obtains the default interface, when `EXT_IF` is not configured, from IPv4 routes, if it fails, it tries again using IPv6 routes, if it fails, an error is displayed. If it succeeds, the obtained interface is used. If `ON_IF` is not defined, the `EXT_IF` value is used.

Some options may use different interfaces if specified by the user.

!!! tip
    
    It is highly recommended to configure `EXT_IF` and `ON_IF` to improve performance
    and make your environment much more stable.
