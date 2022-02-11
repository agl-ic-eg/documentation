---
title: Creating a New Service
---

Services are software running in the background and providing, as their name suggests,
various services to other software: access to specific system hardware, connectivity
management, network servers... They can be split into 2 categories:
- system services: those usually run as a privileged user and make use of shared system
  resources which they should have exclusive access to
- user services: such services run as part of an unprivileged user's session and can
  only be called by said user

# Basic requirements

The only mandatory requirement is that service packages provide a `.service` file
so they can be properly managed by `systemd`. This file must be installed to a specific
location, determined by the service type (system or user):
- `/usr/lib/systemd/system/` for system services
- `/usr/lib/systemd/user/` for user services

Below is an example of a simple user service, running in a graphical session and
therefore requiring a compositor to be already running before the service starts:

```
[Unit]
Requires=agl-compositor.service
After=agl-compositor.service

[Service]
Type=simple
ExecStart=/usr/bin/homescreen
Restart=on-failure

[Install]
WantedBy=agl-session.target
```

The `WantedBy=agl-session.target` indicates the service is part of the default AGL
user session, as mentioned in the [Application Framework](1_Application_Framework/1_Introduction/#user-session-management)
documentation.

The `Restart=on-failure` directive ensures the service will be automatically
restarted by `systemd` in case it crashes.

More details about `systemd` service files can be found in the
[systemd documentation](https://www.freedesktop.org/software/systemd/man/systemd.service.html).

# D-Bus activation

Services can also provide a D-Bus interface. In this case, they need not be started
on system boot (or user session startup in the case of user services) but can be
automatically started only when a client sends a request to the D-Bus name the service
registers.

D-Bus activated services must name their `systemd` service file `dbus-NAME.service`
where `NAME` is the D-Bus name registered by the service. This file must include the
following lines:

```
[Service]
Type=dbus
BusName=NAME
ExecStart=/path/to/executable
```

In addition, they must provide a D-Bus service file named `NAME.service` and installed
to one of the following locations:
- `/usr/share/dbus-1/system-services` for system services
- `/usr/share/dbus-1/services` for user services

The contents of the D-Bus service file must be the following:

```
[D-BUS Service]
Name=NAME
Exec=/path/to/executable
SystemdService=dbus-NAME.service
```

This ensures the service can be safely activated through D-Bus and no conflict will occur
between `systemd` and the D-Bus daemon.

More details about D-Bus activation can be found in the
[D-Bus specification](https://dbus.freedesktop.org/doc/dbus-specification.html#message-bus-starting-services),
under the "Message Bus Starting Services (Activation)" section.

# Services startup

For D-Bus activated services, no additional action is required as those will be automatically
started whenever needed. Other services, however, need a few more steps in order to be
executed on system or session startup.

## System services

System services can take advantage of the Yocto `systemd` class which automates the process of
enabling such services.

1. Ensure the recipe inherits from the `systemd` class:

```
inherit systemd
```

2. Declare the system services that needs to be enabled on boot:

```
SYSTEMD_SERVICE:${PN} = "NAME.service"
```

3. Ensure the `FILES` variable includes the systemd service directory the corresponding
file will be installed to:

```
FILES:${PN} = "\
    ...
    ${systemd_system_unitdir}/* \
"
```

## User services

The `systemd` class doesn't provide an equivalent mechanism for user services. This must
therefore be done manually as part of the package's install process.

1. Make the service a part of the user session:

```
do_install:append() {
    install -d ${D}${systemd_user_unitdir}/agl-session.target.wants
    ln -s ../NAME.service ${D}${systemd_user_unitdir}/agl-session.target.wants/NAME.service
}
```

This ensures `agl-session.target` depends on `NAME.service`, the latter being therefore
automatically started on session creation.

2. Ensure the `FILES` variable includes the systemd service directory the corresponding
file will be installed to:

```
FILES:${PN} = "\
    ...
    ${systemd_user_unitdir}/* \
"
```
