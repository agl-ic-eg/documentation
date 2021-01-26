---
title: Building for virtio
---

Virtio is a standartized interface for implementing virtual I/O devices:

* Russell, Rusty. "virtio: towards a de-facto standard for virtual I/O devices." ACM SIGOPS Operating Systems Review 42.5 (2008): 95-103.

This section describes the steps you need to take to build the AGL demo image
for virtio "platform". Later, the same image can be run on various emulators or
hypervisors that provide virtio devices, for example, QEMU aarch64 emulation on
PC, QEMU/KVM aarch64 virtualization on AGL Reference Hardware, etc.

Below, AGL minimal image and Qt based IVI demo are used as example, but
similiarly one can run HTML5 based demos, cluster demo, or other AGL images.

## 1. Making Sure Your Build Environment is Correct

The
"[Initializing Your Build Environment](./3_Initializing_Your_Build_Environment.md)"
section presented generic information for setting up your build environment
using the `aglsetup.sh` script.
If you are building the AGL demo image for virtio platform, you need to specify
some specific options when you run the script:

```sh
$ source meta-agl/scripts/aglsetup.sh -m virtio-aarch64 -b build-virtio-aarch64 agl-demo
```

The "-m" option specifies the "virtio-aarch64" machine.

The "-b" option sets custom build directory instead of default "build".

The "-f" option might be added to override previously available configuration.
By default, if there were already configuration files in build directory, they
will not be overriden, as result, aglsetup.sh might not have desired effect.

## 2. Using BitBake

This section shows the `bitbake` command used to build the AGL image.

Start the build using the `bitbake` command.

**AGL minimal image :**
The target is `agl-image-minimal`.

```sh
$ bitbake agl-image-minimal
```

**Qt Based IVI demo :**
The target is `agl-demo-platform`.

```sh
$ bitbake agl-demo-platform
```

## 3. Deploying the AGL Demo Image

This subsection describes AGL virtio-aarch64 image deployment under virtio
platform provided by QEMU aarch64 emulator on PC, or QEMU/KVM hypervisor on AGL
Reference Hardware board.

**3.1 QEMU on PC**

If shell from which AGL was built is closed, or new shell is opened, then it is
needed to re-initialize build environment:

```sh
$ source $AGL_TOP/build-virtio-aarch64/agl-init-build-env
```

And further use `runqemu` to boot the image :

```sh
$ runqemu
```

**3.2 QEMU/KVM on AGL Reference Hardware**

Follow these steps to run virtual AGL on bare-metal AGL (AGL-in-AGL) on AGL Reference Hardware board:

  1. Partition eMMC or SD-Card to have two partitions, at least 1 GiB each.
    Actually, can be less but just rounded up to have a nice number.

  2. Flash AGL minimal image root file system to the second partition on SD-Card
    or eMMC.

  3. Build AGL minimal image for AGL Reference Hardware.

    ```sh
    source meta-agl/scripts/aglsetup.sh -m h3ulcb -b build-h3ulcb agl-refhw-h3
    ```

    In `build-h3ulcb/conf/local.conf` add

    ```
    AGL_DEFAULT_IMAGE_FSTYPES = "ext4"
    IMAGE_INSTALL_append = "qemu"
    ```

    CAUTION: Calling aglsetup.sh with "-f" flag will remove above modification
    in "local.conf", so they will be needed to be re-applied.

    Build image:

    ```sh
    bitbake agl-image-minimal
    ```

    Add virtio kernel to the AGL Reference Hardware Linux rootfs:

    ```sh
    cp build-virtio-aarch64/tmp/deploy/images/virtio-aarch64/Image build-h3ulcb/tmp/work/h3ulcb-agl-linux/agl-image-minimal/1.0-r0/rootfs/linux2
    bitbake agl-image-minimal -c image_ext4 -f
    bitbake agl-image-minimal -c image_complete
    ```

    Flash root file system to the first partition on SD-Card or eMMC.

  4. Boot AGL Reference Hardware board using Linux located on the first partition of SD-Card or eMMC.

  5. Run QEMU from Linux 1 command line

    ```sh
    qemu-system-aarch64 \
      -machine virt \
      -cpu cortex-a57 \
      -m 2048 \
      -serial mon:stdio \
      -global virtio-mmio.force-legacy=false \
      -drive id=disk0,file=/dev/mmcblk0p2,if=none,format=raw \
      -device virtio-blk-device,drive=disk0 \
      -object rng-random,filename=/dev/urandom,id=rng0 \
      -device virtio-rng-device,rng=rng0 \
      -nographic \
      -kernel /linux2
      -append 'root=/dev/vda rw mem=2048M'
    ```

    NOTE: mmcblk0p2 above is used for when root file system is flashed on eMMC.
    In case of SD-Card, mmcblk1p2 has to be used.

  6. It is possible to exit from QEMU using monitor commands. Enter "Ctrl+a h" for help.

Know issue: to enable hardware virtualization using KVM, option `-enable-kvm`
could be added to QEMU command line, but it fails with:

```
qemu-system-aarch64: kvm_init_vcpu failed: Invalid argument
```
