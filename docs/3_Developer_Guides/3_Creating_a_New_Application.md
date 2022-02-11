---
title: Creating a New Application
---

Applications are either software designed to perform a specific task during a limited amount of
time, or graphical applications presenting the user with an interface they can interact with.

Such applications are executed by `applaunchd`, the AGL
[application launcher service](1_Application_Framework/2_Application_Startup/).

# Basic requirements

In order to be enumerated by `applaunchd`, applications must provide the a `.desktop` file, as
defined by the [Desktop Entry specification](https://specifications.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html).

The desktop entry file should be installed to `/usr/share/applications` (or the `applications`
sub-directory of any entry present in the `XDG_DATA_DIRS` environment variable) and have a
meaningful name. It is considered good practice to use reverse-DNS notation for the desktop
entry file name, following the recommendations of the [D-Bus specification](https://dbus.freedesktop.org/doc/dbus-specification.html#message-protocol-names):
- this avoids potential name collisions with other desktop entry files
- it makes it easier to enable [D-Bus activation](#d-bus-activation) during the application
  development cycle if needed
- for [graphical applications](#graphical-applications), it ensures the chosen Wayland `app-id`
  will be unique

Such a file must contain at least the following keys:
- `Type`: desktop entry type, must be set to `Application`
- `Name`: application name, as it should be displayed in menus and application launchers
- `Exec`: full path to the main executable

Below is an example of a minimal desktop entry file:

```
[Desktop Entry]
Type=Application
Name=Example Application
Exec=/usr/bin/example-app
```

Graphical applications must also provide an `Icon` entry pointing to the application icon.
The value for this entry must either be the full path to the icon's file or, for icons part
of an existing [icon theme](https://specifications.freedesktop.org/icon-theme-spec/icon-theme-spec-latest.html),
the name of an element of this theme.

In addition, a number of optional fields can be used to change how `applaunchd` will consider the
application:
- `Version`: version of the Desktop Entry Specification the file conforms to, must be `1.5`
- `Hidden`: boolean value, if `true` the application is always ignored by `applaunchd` and
  won't be listed nor executed
- `Terminal`: boolean value, if `true` the application is excluded when requesting the list of
  graphical applications from `applaunchd`
- `DBusActivatable`: boolean value, must be `true` for [D-Bus activated applications](#d-bus-activation)
- `Implements`: list of D-Bus interfaces the application implements, only used for D-Bus activated
  applications.

Finally, graphical applications may also define the `StartupWMClass` key in some cases. Please
refer to the [graphical applications](#graphical-applications) section for more information.

# D-Bus activation

Similarly to [services](2_Creating_a_New_Service.md/#d-bus-activation), applications can
also be activated through D-Bus.

Such applications must name their `.desktop` file after the D-Bus name they register. In addition,
this file must contain the following entries:

```
DBusActivatable=true
Implements=IFACE1;IFACE2;...
```

Where `IFACEn` are the names of the D-Bus interfaces the application implements.

In addition, they must provide a D-Bus service file named `NAME.service` and installed
to `/usr/share/dbus-1/services`.

The contents of the D-Bus service file must be the following:

```
[D-BUS Service]
Name=NAME
Exec=/path/to/executable
```

For example, an application registering the `org.automotivelinux.Example` D-Bus name
and implementing the `org.automotivelinux.Example.Search1` and `org.automotivelinux.Example.Execute1`
interfaces would provide the following files:

* Desktop entry (`/usr/share/applications/org.automotivelinux.Example.desktop`):

```
[Desktop Entry]
Type=Application
Version=1.5
Name=Example Application
Exec=/usr/bin/example-app
Icon=example-icon
Terminal=false
DBusActivatable=true
Implements=org.automotivelinux.Example.Search1;org.automotivelinux.Example.Execute1
```

* D-Bus service file (`/usr/share/dbus-1/services/org.automotivelinux.Example.service`):

```
[D-BUS Service]
Name=org.automotivelinux.Example
Exec=/usr/bin/example-app
```

*Note: in addition to their own D-Bus interface, D-Bus activated applications must also
implement the `org.freedesktop.Application` interface as defined in the
[Desktop Entry specification](https://specifications.freedesktop.org/desktop-entry-spec/1.1/ar01s07.html).*

# Graphical applications

In addition, graphical applications need to comply with a few more requirements:

1. Each application must set a Wayland application ID appropriately as soon as its main window
is created.

2. The `app-id` must be specified in the desktop entry file by adding the following line:

```
StartupWMClass=APP_ID
```

3. The desktop entry file must be named `APP_ID.desktop`.

Doing so will ensure other software can associate the actual `app-id` to the proper application.

*Note: application ID's are set using the [XDG toplevel](https://wayland-book.com/xdg-shell-basics/xdg-toplevel.html)
Wayland interface.*
