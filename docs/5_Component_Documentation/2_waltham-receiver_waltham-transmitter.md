---
title: Waltham receiver/transmitter
---

# Waltham

[Waltham protocol](https://github.com/waltham/waltham) is a IPC library similar
to [Wayland](https://wayland.freedesktop.org), developed with networking in
mind. It operates over TCP sockets, while the wayland protocol only works
locally over a UNIX socket. It retains wayland-esque paradigm, making use of
XMLs to describe the protocol, and it follows an object-oriented design with an
asynchronous architecture.

## Differences from Wayland to Waltham

Some of the differences between Wayland and Waltham are:

* Waltham uses TCP sockets for communication
* Waltham cannot send file descriptors
* Waltham API is minimal and symmetric between server and client sides
* Waltham does not provide an event loop implementation
* The registry implementation is left out of the library, only the interface is
  provided
* No multi-threading support for sharing objects between threads

## Waltham-transmitter and remoting plugin

Surface sharing is not part of Waltham protocol, each system needs to implement
the most efficient way of passing by the buffers from one side to another.  On
AGL, make use of remoting-plugin to enable surface sharing which uses GStreamer
as encoder/decoder. It uses libweston DRM virtual API to grab the buffers, and
then to stream them over the network. The gstreamer pipeline uses UDP while the
input events are communicated with Waltham protocol. The input part is handled
by *waltham-transmitter* plugin which provides an API to create remote
connections and push surfaces over the network and handles remote input. The
act of pushing surface is a misnomer, kept from the older, previous
implementation, and acts a notification mechanism from the transmitter side to
the remote side.

## The receiver client

waltham-receiver application is a wayland demo implementation  which should be
running at the remote side. It is using Waltham protocol to obtain and process
remote input events which handled the transmitter side by the waltham-transmitter
plugin. It creates a similar gstreamer pipeline to process the incoming buffers
and draw and displaying them into a subsurface created by waylandsink.

Contrary to expectations, the waltham receiver is the one that implements the
server side of the Waltham protocol and is capable of displaying the incoming
buffers but also process input events locally and forward them with the help of
the Waltham protocol back at the transmitter side, which in turn will update
the image contents and stream to the receiver, showing the changes caused by
that input. 


     ECU 1                                                               ECU 2
     +---------------------------------------------+                    +--------------------------------------+
     |        +-----------------+                  |                    |                                      |
     |        |     Application |                  |                    |       +-----------+-----------+      |
     |        +-----------------+                  |                    |       | Gstreamer |           |      |
     |                 ^                           |    Buffer   ---------------> (Decode)  |           |      |
     |        wayland  |                    +-------------------/       |       +-----------+           |      |
     |                 v                    |      |    (Ethernet)      |       |     Waltham-receiver  |      |
     |   +----+---------------------+       |      |        -------------------->                       |      |
     |   |    |  Transmitter plugin |<---------------------/            |       +-----------------------+      |
     |   |    |                     |       |      |  Waltham-Protocol  |                     ^                |
     |   |    |---------------------|       |      |                    |             wayland |                |
     |   |    |  Remoting plugin    |-------+      |                    |                     v                |
     |   |    |                     |              |                    |         +---------------------+      |
     |   |    +---------------------+              |                    |         |                     |      |
     |   |                          |              |                    |         |       compositor    |      |
     |   |         compositor       |              |                    |         |                     |      |
     |   +------+-------------------+              |                    |         +----------------+----+      |
     |          |                                  |                    |                          |           |
     |          v                                  |                    |                          v           |
     |   +------------+                            |                    |                    +----------+      |
     |   |  Display   |                            |                    |                    |  Display |      |
     |   |            |                            |                    |                    |          |      |
     |   +------------+                            |                    |                    +----------+      |
     +---------------------------------------------+                    +--------------------------------------+

## Migrating/Placing applications on other outputs

In order to start or place an application on the remoting-ouput output, we can
use `agl-shell-app-id` ini entry for that particular output.

	[transmitter-output]
    name=transmitter-1
    mode=640x720@30
    host=192.168.20.99
    port=5005
    agl-shell-app-id=<APP_ID>

Alternatively, and programmatically, one can use the
[agl-shell-desktop](1_agl-compositor.md#private-extensions) protocol and inform
the compositor that it should migrate it to other, remote outputs.
