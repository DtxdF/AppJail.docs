AppJail is an open-source BSD-3 licensed framework entirely written in sh(1) and C to create isolated, portable and easy to deploy environments using FreeBSD jails that behaves like an application.

Its goals are to simplify life for sysadmins and developers by providing a unified interface that automates the jail workflow by combining the base FreeBSD tools.

AppJail offers simple ways to do complex things.

## Features

* Easy to use.
* Parallel startup (Healthcheckers, Jails & NAT).
* UFS and ZFS support.
* RACCT/RCTL support.
* NAT support.
* Port expose - network port forwarding into jail.
* IPv4 and IPv6 support.
* DHCP and SLAAC support.
* Virtual networks - A jail can be on several virtual networks at the same time.
* Bridge support.
* VNET support
* Deploy your applications much easier using Makejail!
* Netgraph support.
* LinuxJails support.
* Supports thin and thick jails.
* TinyJails - Experimental feature to create a very stripped down jail that is very useful to distribute.
* Startup order control - Using priorities and the boot flag makes management much easier.
* Jail dependency support.
* Initscripts - Make your jails interactive!
* Backup your jails using tarballs or raw images (ZFS only) with a single command.
* Modular structure - each command is a unique file that has its own responsibility in AppJail. This makes AppJail maintenance much easier.
* Table interface - many commands have a table-like interface, which is very familiar to many sysadmin tools.
* No databases - each configuration is separated in each entity (networks, jails, etc.) which makes maintenance much easier.
* Healthcheckers - Monitor your jails and make sure they are healthy!
* Images - Your jail in a single file!
* DEVFS support - Dynamic device management!
* ...
