Although combining `git(1)` and Makejails is useful, it leads to leaving old Makejails that we may no longer use.

```console
# appjail makejail -l
ID                                                                URL
13f4788a856861d05be744697ec17256596b025dc42ca3799483d28c12ed4217  https://github.com/AppJail-makejails/nscde
64d3082bab0f936c42b54ed444246567c58d528db0bb8606efc0680ef7856aaf  https://github.com/AppJail-makejails/python
...
```

Let's suppose we finish developing our python application and we don't need the Makejail for python, so we will remove it.

```console
# appjail makejail -d 64d
[00:00:00] [ debug ] Destroying 64d3082bab0f936c42b54ed444246567c58d528db0bb8606efc0680ef7856aaf ...
```

We can use only the first characters as ID to write less, but if two or more Makejails match, it is necessary to write more characters to make the ID unique.

Makejails are automatically updated by default since the value of `AUTO_GIT_UPDATE` is `1`, but we can manually update a Makejail using `appjail-makejail(1)` `-u`.

```console
# appjail makejail -l
ID                                                                URL
157981f0000f21eaaaac1feae20bf89370a060d89dff4e301806ae708ee21358  https://github.com/AppJail-makejails/go
1682ec25677094ddd737a4f8da60e9d3b0be35535921653d55fa04f532e6be65  https://github.com/AppJail-makejails/nginx
1e973093ef8e324c857c3ddd57f18d1d6367866b47e913139439c67e571a516c  https://github.com/AppJail-makejails/hello
5981af7fe1ecbcbd9421e0e697af0722c50dbca51efd4c7dfb58545eafbb6f90  https://github.com/AppJail-makejails/hello-world
64d3082bab0f936c42b54ed444246567c58d528db0bb8606efc0680ef7856aaf  https://github.com/AppJail-makejails/python
b9188b2095948e43c91ab26a753a7feb10e456d7e5fa15be65d2d9466b90623e  https://github.com/AppJail-makejails/debian
ce6281c4df1de63f37ca6fe890fb4c97bc0bc1d3bd393dd26760aedfde132df6  https://github.com/AppJail-makejails/lxde
# appjail makejail -u 1
[00:00:00] [ info  ] [157981f0000f21eaaaac1feae20bf89370a060d89dff4e301806ae708ee21358] Updating (url = https://github.com/AppJail-makejails/go) ...
[00:00:00] [ info  ] [157981f0000f21eaaaac1feae20bf89370a060d89dff4e301806ae708ee21358] Already up to date.
[00:00:01] [ info  ] [1682ec25677094ddd737a4f8da60e9d3b0be35535921653d55fa04f532e6be65] Updating (url = https://github.com/AppJail-makejails/nginx) ...
[00:00:01] [ info  ] [1682ec25677094ddd737a4f8da60e9d3b0be35535921653d55fa04f532e6be65] Already up to date.
[00:00:01] [ info  ] [1e973093ef8e324c857c3ddd57f18d1d6367866b47e913139439c67e571a516c] Updating (url = https://github.com/AppJail-makejails/hello) ...
[00:00:02] [ info  ] [1e973093ef8e324c857c3ddd57f18d1d6367866b47e913139439c67e571a516c] Updated.
[00:00:02] [ info  ] [1e973093ef8e324c857c3ddd57f18d1d6367866b47e913139439c67e571a516c] Done.
```

To update all Makejails we can use `*`.

```console
# appjail makejail -u '*'
[00:00:00] [ info  ] [5981af7fe1ecbcbd9421e0e697af0722c50dbca51efd4c7dfb58545eafbb6f90] Updating (url = https://github.com/AppJail-makejails/hello-world) ...
[00:00:01] [ info  ] [5981af7fe1ecbcbd9421e0e697af0722c50dbca51efd4c7dfb58545eafbb6f90] Already up to date.
[00:00:01] [ info  ] [157981f0000f21eaaaac1feae20bf89370a060d89dff4e301806ae708ee21358] Updating (url = https://github.com/AppJail-makejails/go) ...
[00:00:02] [ info  ] [157981f0000f21eaaaac1feae20bf89370a060d89dff4e301806ae708ee21358] Already up to date.
[00:00:02] [ info  ] [1682ec25677094ddd737a4f8da60e9d3b0be35535921653d55fa04f532e6be65] Updating (url = https://github.com/AppJail-makejails/nginx) ...
[00:00:02] [ info  ] [1682ec25677094ddd737a4f8da60e9d3b0be35535921653d55fa04f532e6be65] Already up to date.
[00:00:03] [ info  ] [b9188b2095948e43c91ab26a753a7feb10e456d7e5fa15be65d2d9466b90623e] Updating (url = https://github.com/AppJail-makejails/debian) ...
[00:00:03] [ info  ] [b9188b2095948e43c91ab26a753a7feb10e456d7e5fa15be65d2d9466b90623e] Already up to date.
[00:00:03] [ info  ] [ce6281c4df1de63f37ca6fe890fb4c97bc0bc1d3bd393dd26760aedfde132df6] Updating (url = https://github.com/AppJail-makejails/lxde) ...
[00:00:04] [ info  ] [ce6281c4df1de63f37ca6fe890fb4c97bc0bc1d3bd393dd26760aedfde132df6] Already up to date.
[00:00:04] [ info  ] [64d3082bab0f936c42b54ed444246567c58d528db0bb8606efc0680ef7856aaf] Updating (url = https://github.com/AppJail-makejails/python) ...
[00:00:05] [ info  ] [64d3082bab0f936c42b54ed444246567c58d528db0bb8606efc0680ef7856aaf] Already up to date.
[00:00:05] [ info  ] [1e973093ef8e324c857c3ddd57f18d1d6367866b47e913139439c67e571a516c] Updating (url = https://github.com/AppJail-makejails/hello) ...
[00:00:05] [ info  ] [1e973093ef8e324c857c3ddd57f18d1d6367866b47e913139439c67e571a516c] Already up to date.
[00:00:05] [ info  ] [1e973093ef8e324c857c3ddd57f18d1d6367866b47e913139439c67e571a516c] Done.
```
