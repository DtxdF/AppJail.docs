| | AppJail | bastille | pot | iocage | ezjail | |
| --- | --- | --- | --- | --- | --- | --- |
| ZFS support | supported | supported | required | required | supported | ZFS support |
| Language | C, Bourne Shell | Bourne Shell | Rust, Bourne Shell | Python | Bourne Shell | Language |
| Automation | Makejail, Initscripts, Images | Templates | Flavours, Images | Plugins | Flavours | Automation |
| Jail Type | clone, copy, tiny, thin, thick, empty, linux+debootstrap | thin, thick, vnet, Linux, empty | thick | clone, basejail, template, empty, thick | basejail | Jail Type |
| VNET | Supported | Supported | Supported | Supported | Not Supported | VNET |
| Dynamic firewall | Yes | Requires a loopback interface | Yes | No | No | Dynamic firewall |
| Resource control | Full support | Yes, but it does not support statistics and all `rctl(8)` actions and does not support actions by rule | Basic: CPU and memory only | Legacy only | Not Supported | Resource control |
| CPU Sets | Yes | No | Yes | Yes | Yes | CPU Sets |
| IPv6 support | Yes (+SLAAC) | Yes | Yes | Yes | Yes | IPv6 support |
| Linux containers | Yes  | Yes | ?? | Yes | ?? | Linux containers |
| Dynamic DEVFS Ruleset Management | Yes | No | No | No | No | Dynamic DEVFS Ruleset Management |
| OCI | supported | No | No | No | No | OCI |
| Network management | Virtual networks, Bridges  | No | Subnet, requires `sysutils/potnet` | No | No | Network management |
| Jail dependency | Yes | No | Yes | Yes | No | Jail dependency |
| Supervisor | Yes (Healthcheckers) | No | No | No | No | Supervisor |
| Log management | Yes | No | No | No | No | Log management |
| Parse `jail.conf(5)` file for syntax errors | Yes | Not supported | Yes | Yes | Yes | Parse `jail.conf(5)` file for syntax errors |
| Volume management | Yes | No | Basic, Only supported when using the `fscomp` feature | Basic | No | Volume management |
| Parallel startup | Yes (Healthcheckers, jails & NAT) | No | No | No | No | Parallel startup |
| Netgraph | Yes  | No | No | No | No | Netgraph |
| Startup order control | Yes  | Yes, but don't support priorities | Yes, but don't support priorities | Yes | Yes, using `rcorder(8)` | Startup order control |
| X11 support | Yes  | No | No | No | No | X11 support |
| import/export | Yes  | Yes | Yes | Yes | Yes | import/export |
