---
title: Usage
---

## Installation

The afb-helpers library is integrated by default in the AGL SDK since the Guppy
version (>=7) and is also available as a package for the AGL supported linux
distributions.

You could find the SDK build from Yocto which embed the afb-helpers library
here:

- **x86** : [qemux86-64](https://download.automotivelinux.org/AGL/snapshots/master/latest/qemux86-64/deploy/sdk/poky-agl-glibc-x86_64-agl-demo-platform-crosssdk-corei7-64-qemux86-64-toolchain-10.90.0+snapshot.sh)

- **ARM 32 bit** : [qemuarm](https://download.automotivelinux.org/AGL/snapshots/master/latest/qemuarm/deploy/sdk/poky-agl-glibc-x86_64-agl-demo-platform-crosssdk-armv7vet2hf-neon-vfpv4-qemuarm-toolchain-10.90.0+snapshot.sh)

- **AARCH64 - ARM 64bit** : [qemuarm64](https://download.automotivelinux.org/AGL/snapshots/master/latest/qemuarm64/deploy/sdk/poky-agl-glibc-x86_64-agl-demo-platform-crosssdk-aarch64-qemuarm64-toolchain-10.90.0+snapshot.sh)

Then use your package manager to install the library.

### OpenSuse

```bash
sudo zypper ref
sudo zypper install agl-libafb-helpers-devel
```

### Fedora

```bash
sudo dnf ref
sudo dnf install agl-libafb-helpers-devel
```

### Ubuntu/Debian

```bash
sudo apt-get update
sudo apt-get install agl-libafb-helpers-dev
```

## (Optional) Remove the git submodule version

If you already use the afb-helpers component but using the submodule version
then you have to get rid of it to be sure to link and use the library version.
To do so, you have to do the following:

* Deinitialize the submodule using `git`

```bash
# This example assumes that the git submodule is named app-afb-helpers-submodule
# and is located at your root project repository.
git submodule deinit app-afb-helpers-submodule
```

* Remove the submodule relatives lines from the `.gitmodules` file

```bash
vim .gitmodules
```

* Remove the `afb-helpers` target link from any CMake target you specified.
 Those lines look like:

```bash
TARGET_LINK_LIBRARIES(${TARGET_NAME}
    afb-helpers # REMOVE THIS LINE
    ${link_libraries}
    )
```

## Add the libafb-helpers as a static library to your binding

In your `config.cmake` file, add a dependency to the controller library, i.e:

```cmake
set(PKG_REQUIRED_LIST
	json-c
	afb-daemon
	afb-helpers --> this is the afb-helpers library dependency name.
)
```