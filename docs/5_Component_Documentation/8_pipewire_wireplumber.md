---
title: PipeWire / WirePlumber
---

# PipeWire / WirePlumber

## Overview

AGL uses the PipeWire daemon service to provide audio playback and capture capabilities.
PipeWire is accompanied by a secondary service, WirePlumber (also referred to as the
*session manager*), which provides policy management, device discovery, configuration and more.

Applications can connect to the PipeWire service through its UNIX socket, by using either the
*libpipewire* or *libwireplumber* libraries as a front-end to that socket.

Upstream documentation for these components can be found at the links below:

- [PipeWire documentation](https://docs.pipewire.org/)

- [WirePlumber documentation](https://pipewire.pages.freedesktop.org/wireplumber/) 

- [PipeWire Wiki](https://gitlab.freedesktop.org/pipewire/pipewire/-/wikis/home)

## APIs

### libpipewire

The main entry point for applications to access the audio system is the API offered by
*libpipewire*. The functionality offered by *libpipewire* is vast and it is beyond the
scope of this document to describe it all.

For playback and capture, applications should use *struct pw_stream* and its associated methods. 
See [PipeWire: Tutorial - Part 4: Playing a tone](https://docs.pipewire.org/page_tutorial4.html)
for a starting point.

### GStreamer (Recommended)

For convenience, applications that use GStreamer can use the PipeWire GStreamer elements to 
plug the functionality offered by *struct pw_stream* directly in the GStreamer pipeline. These 
elements are called *pipewiresrc* and *pipewiresink*

Example:

```shell
> gst-launch-1.0 audiotestsrc ! pipewiresink
```

Through these elements, it is possible to specify the application role by setting it in the
*stream-properties* property of the element, as shown below:

```shell
> gst-launch-1.0 audiotestsrc ! pipewiresink stream-properties="p,media.role=Multimedia""
```

or, in the C API:

```c
gst_util_set_object_arg (sink, "stream-properties", "p,media.role=Multimedia");
```

Of course, it is also possible to use *alsasink* and *alsasrc* and route audio through the
virtual ALSA device that is mentioned below. This is also the default behavior of *playbin*
and similar auto-plugging elements, because the PipeWire GStreamer elements are not autoplugged
(this may change in a future version).

### ALSA

PipeWire offers a virtual ALSA device (called *pipewire*) that redirects audio to PipeWire
through an ALSA PCM plugin. This device is the default one, so unless you explicitly specify
a device in your ALSA client application, audio will go through PipeWire instead.

Example:

```shell
> aplay sound.wav  # the default device is 'pipewire'
> aplay -D pipewire sound.wav
```

In order to specify the application role while using the ALSA compatibility device, pass the role
as a device parameter like this:

```shell
> aplay -D pipewire:ROLE=Navigation turnleft.wav
```

### Audiomixer service

See the separate [agl-service-audiomixer](https://git.automotivelinux.org/apps/agl-service-audiomixer/about/) documentation.

### libwireplumber

The WirePlumber library provides API that wraps libpipewire and makes it easier to work with 
when you are writing control applications, such as a volume mixer. The audiomixer service is in 
fact implemented using *libwireplumber*.

WirePlumber also provides support for lua-based scripting. Standalone scripts, executed with the 
*wpexec* tool, may be used  as a means to rapidly make use of the API provided by *libwireplumber*

## Tools

* **wpctl**: allows inspecting the devices, choosing which source & sink are the default ones 
  and allows volume/mute adjustments to be made on the command line. Try `wpctl status` and
  `wpctl help` to get started with it

* **wpexec**: allows running wireplumber lua scripts standalone, which is useful to implement
  custom scripts to interact with PipeWire

* **pw-cli**: this is the main tool for interacting with pipewire directly

* **pw-dump**: dumps all the objects in the pipewire graph to a big JSON. The output of this
  tool is very useful to include in bug reports. It is also suitable for implementing scripts
  that parse information with jq

* **pw-dot** is a useful debug tool that dumps the objects in a dot graph for easy visualization

* **pw-cat / pw-play / pw-record**: This is a set of tools similar to aplay/arecord, for simple
  audio operations

* **pw-top**: This is a performance measurement tool that shows realtime timing information
  about the audio pipeline. Before running this tool, you will need to uncomment the loading
  of "libpipewire-module-profiler" in /etc/pipewire/pipewire.conf and restart pipewire

## Systemd Integration

The PipeWire service, `pipewire.service`, is activated on demand, via systemd socket activation,
by `pipewire.socket`. The WirePlumber service, `wireplumber.service`, is bound to the pipewire
service and therefore started and stopped together with the PipeWire service.

If you wish to manually stop or restart both services, you can do so by using *systemctl*,
operating on the *.socket* unit:

```shell
> systemctl restart pipewire.socket
> systemctl stop pipewire.socket
```

## Debugging

The PipeWire daemon can be configured to be more verbose by editing
**/etc/pipewire/pipewire.conf** and setting `log.level=n` (n=0-5) in section
`context.properties`.

Similarly, the WirePlumber daemon can be configured to be more verbose by editing
**/etc/wireplumber/wireplumber.conf** and setting `log.level=n` (n=0-5) in section
`context.properties`.

All messages will be available in the systemd journal, inspectable with journalctl.

For applications, at runtime, `PIPEWIRE_DEBUG` can be set to a value between 0 and 5, 
with 5 being the most verbose and 0 the least verbose.

For applications that use *libwireplumber* the equivalent environment variable is
`WIREPLUMBER_DEBUG`, which also takes values between 0 and 5.

The default debug level for the daemons is 2, while for applications it's 0
(with few exceptions).

More information is also available on 
[WirePlumber's documentation on debug logging](https://pipewire.pages.freedesktop.org/wireplumber/daemon-logging.html)
