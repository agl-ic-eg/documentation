---
title: Introduction
---

# Foreword

The AGL Application Framework is nothing new. However, the implementation used up until
the `lamprey` release has been retired starting with the `marlin` release and replaced
by a redesigned Application Framework one. However, this new implementation isn't a 1:1
replacement, and as such it doesn't provide all of the features of the previous
Application Framework. Some of those will be added back over time, others have been
discarded in favor of more modern and/or widely-used alternatives.

# Introduction

As a provider of an integrated solution to build up on, AGL needs to define a reliable
and well-specified method for managing the deployment and integration of applications
and services, as well as the way they can interact with the rest of the system.

This is achieved by providing a common set of rules and components, known as the
Application Framework. By ensuring conformity to those rules, application developers
can have a good understanding of the requirements for creating and packaging
applications targeting AGL-based systems. Likewise, system developers and integrators
have a clear path for including such applications in AGL-based products.

The Application Framework's scope extends to the following areas:
- system services integration and lifecycle management
- user session management, including user-level applications and services lifecycle
  management
- inter-process communication

In order to be as simple as possible and avoid any unneded custom implementation, the
Application Framework relies mainly on third-party technologies and/or software
components, most of those being maintained under the
[freedesktop.org](https://www.freedesktop.org) umbrella. Those include:
- [systemd](https://www.freedesktop.org/wiki/Software/systemd/): system and services
  management
- [D-Bus](https://www.freedesktop.org/wiki/Software/dbus/): inter-process communication
- [Desktop Entry specification](https://www.freedesktop.org/wiki/Specifications/desktop-entry-spec/):
  application enumeration and startup

AGL also provides reference implementations whenever possible and relevant, located in
the [meta-agl](../6_AGL_Layers/2_meta-agl/) layer under `meta-app-framework`. At the
moment, the Application Framework contains 2 such components:
- `agl-session`: `systemd` unit files for user sessions management
- `applaunchd`: application launcher service

# Services management

Both system and user services are managed by `systemd`, which provides a number of
important features, such as dependency management or service monitoring: when starting
a service, `systemd` will ensure any other units this service depends on are available,
and otherwise start those dependencies. Similarly, `systemd` can automatically restart
a crashed service, ensuring minimal downtime.

`systemd` also provides an efficient first layer of security through its
[sandboxing](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#Sandboxing)
and other security-related options.

It is also well integrated with D-Bus and can be used for a more fine-grained control
over D-Bus activated services: by delegating the actual service startup to `systemd`,
developers can take advantage of some of its advanced features, allowing for improved
reliability and security.

Each service should be represented by a `systemd` unit file installed to the appropriate
location. More details can be obtained from the [Creating a New Service](../2_Creating_a_New_Service/)
document.

# User session management

Similarly, user sessions and the services they rely on are also managed by `systemd`.

AGL provides 2 `systemd` units:

1. `agl-session@.service` is a template system service for managing user sessions; it
takes a username or UID as a parameter, creating a session for the desired user.
Instanciating this service can be achieved by enabling `agl-session@USER.service`, for
example by executing the following command on a running system:

```
$ systemctl enable agl-session@USER.service
```

By default, AGL enables this service as `agl-session@agl-driver.service`, running as
user `agl-driver`.

*Note: while you can create sessions for as many users as needed, only one instance of
`agl-session@.service` is allowed per user.*

2. `agl-session.target` is a user target for managing user services and their
dependencies. It is started by `agl-session@.service`.

By default, `agl-compositor` is part of this target. It is therefore automatically
started for user `agl-driver`.

Any other service needed as part of the user session should similarly depend on this
target by appending the following lines to their unit file:

```
[Install]
WantedBy=agl-session.target
```

# Inter-process communication

In order to provide a "standard", language-independent IPC mechanism and avoid the need
for maintaining custom bindings for each programming language to be used on top of AGL,
the Application Framework promotes the use of [D-Bus](https://www.freedesktop.org/wiki/Software/dbus/)
as the preferred way for applications to interact with services.

Most services already included in AGL provide one or several D-Bus interfaces, and can
therefore interact with D-Bus capable applications and services without requiring any
additional component. Those services include, among others:
- [ConnMan](https://git.kernel.org/pub/scm/network/connman/connman.git/): network connectivity
- [BlueZ](http://www.bluez.org/): Bluetooth connectivity
- [oFono](https://git.kernel.org/pub/scm/network/ofono/ofono.git): telephony and modem
  management
- [GeoClue](https://gitlab.freedesktop.org/geoclue/geoclue/-/wikis/home): geolocation

# Application launcher service

As mentioned above, the Application Framework follows the guidelines of the
[Desktop Entry specification](https://www.freedesktop.org/wiki/Specifications/desktop-entry-spec/)
for application enumeration and startup.

As no simple reference implementation exists for this part of the specification, AGL
provides an application launcher service named `applaunchd`. This service is part of the
default user session, and as such is automatically started on session startup. It can
therefore be considered always available.

`applaunchd` enumerates applications installed on the system and provides a D-bus
interface for services and applications to:
- query the list of available applications
- request the startup and/or activation of a specific application
- be notified when applications are started or terminated

`applaunchd` is described with more details in [the following document](2_Application_Startup/).
