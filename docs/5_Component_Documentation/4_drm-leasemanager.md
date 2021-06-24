---
title: DRM lease manager
---

# DRM Lease Manager

## Overview

The DRM Lease Manager is used in AGL to allocate display controller outputs to different processes. Each process has direct access to its allocated output via the kernel DRM interface, and is prevented from accessing any other outputs managed by the display controller.

The display controller partitioning is made possible by using the kernel DRM Lease feature.  For more information on the DRM lease functionality, please see the Linux kernel [DRM userspace API documentation](https://www.kernel.org/doc/html/latest/gpu/drm-uapi.html).


## Usage

### Building DRM Lease manager and sample clients

Enable the  `agl-drm-lease` AGL feature when setting up your build environment
with aglsetup.sh.

This will add the drm-lease-manager package to the image, and will add DRM
lease support to the following packages:

* weston
* kmscube

kmscube is not included in the image by default. To add the package to the
image, add the following to your local.conf

```
IMAGE_INSTALL_append = " kmscube"
```

### Starting the DRM lease manager

The drm-lease-manager must be the only process to directly open the DRM device.
Shut down any running window systems (eg. weston or agl-compositor) and run:

```shell
  systemctl start drm-lease-manager
```

This will create a lease for each output connection on the platform.
The name of each lease will be in the form of `card0-<output name>`
 (eg. `card0-LVDS-1` or `card0-HDMI-A-1`)

**Note**: `drm-lease-manager` can be started on a different display controller (i.e. not `card0`) by modifying the command line in the systemd unit file.

Run the help command for details.

```shell
  drm-lease-manager --help
```

### Running weston

weston can be started on any available DRM lease by running with the
`--drm-lease=<lease name>` option.

```shell
  weston --drm-lease=card0-HDMI-A-1
```

### Running kmscube sample
With the `drm-lease-manager` running, `kmscube` can display on any available
lease by launching it with the `-L -D<lease name>` options.

```shell
  kmscube -L -Dcard0-HDMI-A-1
```

Multiple weston or kmscube instances (one per DRM lease) can be started at the same time.

## Adding support to your application

The DRM Lease Manager packages includes a client library (libdlmclient) to request access to the DRM leases it creates.  The client library provides file descriptors that can be used as if they were created by directly opening the underlying DRM device.

Clients that want _direct access_ to the DRM device can use this library to do so.  Compositor (agl-compositor or weston) client applications do not need to use this interface.  


To use the DRM leases, clients only need to replace their calls to
`drmOpen()` and `drmClose()` with the appropriate libdlmclient API calls.

### libdlmclient API usage

_Error handling has been omitted for brevity and clarity of examples._

#### Requesting a lease from the DRM Lease Manager

```c
  struct dlm_lease *lease = dlm_get_lease("card0-HDMI-A-1");
  int drm_device_fd = dlm_lease_fd(lease);
```

`drm_device_fd` can now be used to access the DRM device

#### Releasing a lease

```c
  dlm_release_lease(lease);
```

**Note: `drm_device_fd` is not usable after calling `dlm_release_lease()`**

## Runtime directory
A runtime directory under `/var` is used to keep the Unix Domain sockets that the drm-lease-manager and clients to communicate with each other.

The default path is `/var/run/drm-lease-manager`, but can be changed by setting the `DLM_RUNTIME_PATH` environment variable.
