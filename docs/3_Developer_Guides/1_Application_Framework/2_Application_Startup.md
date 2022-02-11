---
title: Application Startup
---

# Introduction

At system runtime, it may be necessary for applications to start other applications
on demand. Such actions can be executed in reaction to a user request, or they may
be needed to perform a specific task.

In order to do so, running applications and services need an established way of
discovering installed applications and executing those. The
[Desktop Entry specification](https://specifications.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html)
defines how applications can be discovered by using `.desktop` files, but there's no
simple reference implementation for this function.

In order to provide a language-independent interface for applications and service to
use, AGL includes `applaunchd`, a user service part of the default session.

*Note: as mentioned [previously](1_Introduction/), services are managed using `systemd`
and are therefore not in the scope of this document.*

# Application launcher service

The purpose of `applaunchd` is to enumerate applications available on the system and
provide a way for other applications to query this list and start those on demand.
It is also able to notify clients of the startup and termination of applications it
manages.

To that effect, `applaunchd` provides a D-Bus interface other applications can use
in order to execute those actions.

*Note: `applaunchd` will only send notifications for applications it started; it isn't
aware of applications started by other means (`systemd`, direct executable call...),
and therefore can't send notifications for those.*

## Application discovery

On startup, `applaunchd` inspects all `.desktop` files present under the `applications/`
subfolder of any element of the `XDG_DATA_DIRS` environment variable, ignoring all entries
containing either the `NoDisplay=true` or `Hidden=true` lines.

It then looks for the following keys:
- `Terminal`
- `DBusActivatable`

If the desktop entry file contains the `Terminal` key set to `true`, then the application
is marked as a non-graphical one. As such, it won't be included in the applications list
if the client requests only graphical applications.

If `DBusActivatable` is set to `true`, then the application is marked as D-Bus activated.
Additionally, `applaunchd` will search for a corresponding D-Bus service file in case this
line is missing. This is a workaround allowing D-Bus activated applications providing
an incomplete desktop entry file (i.e missing the `DBusActivatable` key) to be
identified as such.

### Requirements for D-Bus activation

`applaunchd` will always start D-Bus activatable applications using D-Bus activation
instead of executing the command line stated in the desktop entry file.

This is handled by calling the `Activate` method of the
[org.freedesktop.Application](https://specifications.freedesktop.org/desktop-entry-spec/1.1/ar01s07.html)
interface with an empty argument.

As a consequence, all D-Bus activatable applications **must** implement this D-Bus
interface.

## Application identifiers

Each application is identified by a unique Application ID. Although this ID can be
any valid string, it is highly recommended to use the "reverse DNS" convention in order
to avoid potential name collisions and ease D-Bus integration.

The application ID is set in the desktop entry file itself for
[graphical applications](../3_Creating_a_New_Application/#graphical-applications):
it is the value of the `StartupWMClass` field, which must be identical to the `app-id`
advertised through the Wayland XDG toplevel protocol. In case this field is missing
(as is usually the case for non-graphical application), the application ID will be the
desktop entry file name, stripped from its `.desktop` extension.

## D-Bus interface

The `applaunchd` D-Bus interface is named `org.automotivelinux.AppLaunch`. The object
path for `applaunchd` is `/org/automotivelinux/AppLaunch`. The interface provides methods
for the following actions:
- retrieve the list of available applications; the client can choose to retrieve all
  available applications, or only those suitable for a graphical environment
- request an application to be started

Moreover, signals are available for clients to be notified when an application has
successfully started or its execution terminated.

### Applications list

The `listApplications` method allows clients to retrieve the list of available applications.
It takes one boolean argument named `graphical`:
- if set to `true`, only applications suitable for graphical environments are returned
- otherwise, the list contains all applications

This method returns an array of variants (type `av`), each element being a structure made up
of 3 strings (type `(sss)`):
- the application ID
- the application's displayed name
- the full path to the application icon file (or an empty string if no icon was specified in
  the application's desktop entry file)

### Application startup request

Applications can be started by using the `start` method, passing the corresponding application
ID as the only argument. This method doesn't return any data.

If the application is already running, `applaunchd` won't start another instance, but instead
emit a `started` signal to notify clients the application is ready.

### Status notifications

The `org.automotivelinux.AppLaunch` interface provides 2 signals clients can connect to:
- `started` indicates an application has started
    - for D-Bus activated applications, it is emitted upon successful completion of the
      call to the `Activate` method of the `org.freedesktop.Application` interface
    - for other applications, this signal is emitted as soon as the child process has been
      successfully created
- `terminated` is emitted when an application quits

Both signals have an additional argument named `appid`, containing the ID of the application
affected by the event.

As mentioned above, the `started` signal is also emitted if `applaunchd` receives a request to
start an already running application. This can be useful, for example, when switching between
graphical applications:
- the application switcher doesn't need to track the state of each application; instead, it can
  simply send a `start` request to `applaunchd` every time the user requests to switch to another
  application
- the desktop environment then receives the `started` signal, indicating it should activate the
  window with the corresponding `app-id`

## Testing

`applaunchd` can be manually tested using the `gdbus` command-line tool:

1. Query the application list (graphical applications only):

```
$ gdbus call --session --dest "org.automotivelinux.AppLaunch" \
                       --object-path "/org/automotivelinux/AppLaunch" \
                       --method "org.automotivelinux.AppLaunch.listApplications" \
                       true
```

This command will output something similar to what follows:

```
([<('navigation', 'Navigation', '/usr/share/icons/hicolor/scalable/navigation.svg')>,
 <('settings', 'Settings', '/usr/share/icons/hicolor/scalable/settings.svg')>,
 <('dashboard', 'Dashboard', '/usr/share/icons/hicolor/scalable/dashboard.svg')>,
 <('hvac', 'HVAC', '/usr/share/icons/hicolor/scalable/hvac.svg')>,
 <('org.freedesktop.weston.wayland-terminal', 'Weston Terminal', '/usr/share/icons/Adwaita/scalable/apps/utilities-terminal-symbolic.svg')>],)
```

2. Request startup of the `org.freedesktop.weston.wayland-terminal` application:

```
$ gdbus call --session --dest "org.automotivelinux.AppLaunch" \
                       --object-path "/org/automotivelinux/AppLaunch" \
                       --method "org.automotivelinux.AppLaunch.start" \
                       "org.freedesktop.weston.wayland-terminal"
```

3. Monitor signals emitted by `applaunchd`:

```
$ gdbus monitor --session --dest "org.automotivelinux.AppLaunch"
```

This results in the following output when starting, then exiting, the
`org.freedesktop.weston.wayland-terminal` application:

```
Monitoring signals from all objects owned by org.automotivelinux.AppLaunch
The name org.automotivelinux.AppLaunch is owned by :1.4
/org/automotivelinux/AppLaunch: org.automotivelinux.AppLaunch.started ('org.freedesktop.weston.wayland-terminal',)
/org/automotivelinux/AppLaunch: org.automotivelinux.AppLaunch.terminated ('org.freedesktop.weston.wayland-terminal',)
```
