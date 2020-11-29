---
title: Installing the binder on a desktop
---

Packages of the ***binder*** (afb-daemon)exist
for common desktop linux distributions.

- Fedora
- Ubuntu
- Debian
- Suse

Installing the development package of the ***binder***
allows to write ***bindings*** that runs on the desktop
computer of the developer.

It is very convenient to quickly write and debug a binding.

## Retrieving compiling option with pkg-config

The ***binder*** afb-daemon provides a configuration
file for **pkg-config**.
Typing the command

```bash
pkg-config --cflags afb-daemon
```

Print flags use for compilation:

```bash
$ pkg-config --cflags afb-daemon
-I/opt/local/include -I/usr/include/json-c
```

For linking, you should use

```bash
$ pkg-config --libs afb-daemon
-ljson-c
```

It automatically includes the dependency to json-c.
This is activated through **Requires** keyword in pkg-config.
