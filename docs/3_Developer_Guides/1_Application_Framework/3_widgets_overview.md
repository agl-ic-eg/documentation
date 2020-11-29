---
title: Overview of widgets
---


# Tools for managing widgets

This project includes tools for managing widgets.
These tools are:

- ***wgtpkg-info***: command line tool to display
  informations about a widget file.

- ***wgtpkg-install***: command line tool to
  install a widget file.

- ***wgtpkg-pack***: command line tool to create
  a widget file from a widget directory.

- ***wgtpkg-sign***: command line tool to add a signature
  to a widget directory.

For all these commands, a tiny help is available with
options **-h** or **--help**.

There is no tool for unpacking a widget.
For doing such operation, you can use the command **unzip**.

To list the files of a widget:

```bash
unzip -l WIDGET
```

To extract a widget in some directory:

```bash
unzip WIDGET -d DIRECTORY
```

*Note: that DIRECTORY will be created if needed*.

## Getting data about a widget file

The command **wgtpkg-info** opens a widget file, reads its **config.xml**
file and displays its content in a human readable way.

## Signing and packing widget

### Signing

To sign a widget, you need a private key and its certificate.

The tool **wgtpkg-sign** creates or replace a signature file in
the directory of the widget BEFORE its packaging.

There are two types of signature files: author and distributor.

Example 1: add an author signature

```bash
wgtpkg-sign -a -k me.key.pem -c me.cert.pem DIRECTORY
```

Example 2: add a distributor signature

```bash
wgtpkg-sign -k authority.key.pem -c authority.cert.pem DIRECTORY
```

### Packing

This operation can be done using the command **zip** but
we provide the tool **wgtpkg-pack** that may add checking.

Example:

```bash
wgtpkg-pack DIRECTORY -o file.wgt
```

## Writing a widget

### App directory organization

There are few directories that are by default used in the binder. At the root
directory of the widget folder, here are the directories that could be used:

- *lib* : default directory where lies external libraries needed for
 the binding. Binding could be stored here too or in another directory of your
 choice.
- *htdocs* : root HTTP directory if binding has web UI.

### The steps for writing a widget

1. make your application
2. create its configuration file **config.xml**
3. sign it
4. pack it

Fairly easy, no?

## Organization of directory of applications

### Directory where are stored applications

Applications can be installed in different places:

- the system itself, extension device.

On a phone application are typically installed on the sd card.

This translates to:

- /var/local/lib/afm/applications

From here this path is referenced as: "APPDIR".

The main path for applications is: APPDIR/PKGID/VER.

Where:

- APPDIR is as defined above
- PKGID is a directory whose name is the package identifier
- VER is the version of the package MAJOR.MINOR

This organization has the advantage to allow several versions
to leave together.
This is needed for some good reasons (rolling back) and also for less good reasons (user habits).

### Identity of installed files

All files are installed as user "afm" and group "afm".
All files have rw(x) for user and r-(x) for group and others.

This allows every user to read every file.

### Labeling the directories of applications

The data of a user are in its directory and are labelled by the security-manager using the application labels.

[widgets]:          http://www.w3.org/TR/widgets                                    "Packaged Web Apps"
[widgets-digsig]:   http://www.w3.org/TR/widgets-digsig                             "XML Digital Signatures for Widgets"
[app-manifest]:     http://www.w3.org/TR/appmanifest                                "Web App Manifest"
[meta-intel]:       https://github.com/01org/meta-intel-iot-security                "A collection of layers providing security technologies"
[widgets]:          http://www.w3.org/TR/widgets                                    "Packaged Web Apps"
[widgets-digsig]:   http://www.w3.org/TR/widgets-digsig                             "XML Digital Signatures for Widgets"
[libxml2]:          http://xmlsoft.org/html/index.html                              "libxml2"
[openssl]:          https://www.openssl.org                                         "OpenSSL"
[xmlsec]:           https://www.aleksey.com/xmlsec                                  "XMLSec"
[json-c]:           https://github.com/json-c/json-c                                "JSON-c"
[d-bus]:            http://www.freedesktop.org/wiki/Software/dbus                   "D-Bus"
[libzip]:           http://www.nih.at/libzip                                        "libzip"
[cmake]:            https://cmake.org                                               "CMake"
[security-manager]: https://wiki.tizen.org/wiki/Security/Tizen_3.X_Security_Manager "Security-Manager"
[app-manifest]:     http://www.w3.org/TR/appmanifest                                "Web App Manifest"
[tizen-security]:   https://wiki.tizen.org/wiki/Security                            "Tizen security home page"
[tizen-secu-3]:     https://wiki.tizen.org/wiki/Security/Tizen_3.X_Overview         "Tizen 3 security overview"
