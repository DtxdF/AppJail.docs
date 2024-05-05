The previous section describes how to use Makejail to deploy a jail, but it uses fixed strings. Fixed strings introduce some problems, especially with network configurations, package versions, etc.

Makejail solves this by using arguments to change things dynamically.

`php/Makejail` is a good example when we must use arguments.

**php/Makejail**:

```
ARG php_ver=80

PKG php${php_ver}
SYSRC php_fpm_enable=YES

COPY --verbose usr

CMD ln -s /usr/local/etc/php.ini-production /usr/local/etc/php.ini

SERVICE php-fpm start
```

Now, the user can install a PHP version such as `www/php80`, `www/php81`, `www/php82` or any available version.

The default value for `php_ver` is `80`, but it can be changed simply by using `--php_ver` in `appjail-makejail(1)`.

```sh
appjail makejail -f Makejail -j php -- --php_ver 81
```

Another good example is the `options/network.makejail` file that uses the `web` network. This network probably does not exist when the end user runs the Makejail, so a good option is to use this argument as non-optional.

**options/network.makejail**:

```
ARG network

OPTION virtualnet=${network}:${APPJAIL_JAILNAME} default
OPTION nat
```

Arguments make the Makejail dynamic, but they can only be used for the current stage. There are some cases where we need to use dynamic values in all stages.

Build arguments can be used to replace values regardless of the current stage. `appjail-makejail(1)` uses them after compiling all Makejails.

To use build arguments use a syntax like `%{name[: | !][value]}` on any line, although it is recommended to use them on the first lines.

* `name`: It is recommended to write it in upper case.
* `: | !`: If the `:` character is used, `value` will be used as default value when not set by the user. If the `!` is used, `appjail-makejail(1)` will complain when the build argument is not set by the user, or in other words, this causes the build argument to be required and `value` is used as a description.

A good example is when using Python. The Python executable is named `pythonx.y`, where `x` is the major version and `y` is the minor version. If we need to use Python in some stages for an application, we must use such numbers. Of course, the application must be written to use the specific python version and there are better options for the above example such as using shegbang.

`appjail-makejail(1)` uses the name of the build arguments to search and replace in the same way as in the `REPLACE` command.

Arguments can use other arguments as a value, but the `}` character must be escaped using `\`

```
%{PYTHON_EXECUTABLE:python%{PYTHON_MAJOR\}.%{PYTHON_MINOR\}}
%{PYTHON_MAJOR:3}
%{PYTHON_MINOR:9}
```

The example above shows what arguments look like. When `appjail-makejail(1)` sees a line that uses the `%{PYTHON_EXECUTABLE}` (or any other) build argument, it replaces it with its value. For example:

```
CMD %{PYTHON_EXECUTABLE}
```

The first step in the Python Makejail is to replace `%{PYTHON_EXECUTABLE}`, so the above example will become:

```
CMD python%{PYTHON_MAJOR}.%{PYTHON_MINOR}
```

The second step is to replace `%{PYTHON_MAJOR}`:

```
CMD python3.%{PYTHON_MINOR}
```

The third and final step is to replace `%{PYTHON_MINOR}`:

```
CMD python3.9
```

The process is the same for the rest of the arguments.

If we create a Makejail for Python that uses Build Arguments, we will probably need `VAR` and `RAW` instructions for some checks.

```
VAR --noexpand python_major=%{PYTHON_MAJOR}
VAR --noexpand python_minor=%{PYTHON_MINOR}

RAW if ! echo "${python_major}" | grep -Eq '^[23]$'; then
RAW     echo "MAJOR VALID VERSIONS: 2, 3" >&2
RAW     exit 1
RAW fi

RAW if ! echo "${python_minor}" | grep -Eq '^(7|8|9|10|11)$'; then
RAW     echo "MINOR VALID VERSIONS: 7, 8, 9, 10, 11" >&2
RAW     exit 1
RAW fi

RAW if [ ${python_major} -eq 2 ]; then
RAW     if [ ${python_minor} -ne 7 ]; then
RAW             echo "MINOR VALID VERSION FOR 2: 7" >&2
RAW             exit 1
RAW     fi
RAW fi

PKG python${python_major}${python_minor}
```

The Makejail uses `VAR` with `--noexpand` to escape the `$` character (see [VAR](instructions.md#var) for more details).

The rest of the code shows what this Makejail uses. `RAW` to perform some useful checks. When all the checks are correct, python installs using variables instead of build arguments. All checks for build arguments should be performed at build stage unless there is a good reason to perform them at another stage.

!!! note

    The build arguments can also be named `Macro Variables`.
