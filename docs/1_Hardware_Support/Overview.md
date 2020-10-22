---
title: Supported Boards
---

The following table briefs about the various hardware platforms, supported by AGL :

**NOTE:** Further information about AGL Distribution available at [AGL wiki](https://wiki.automotivelinux.org/agl-distro).

### AGL Reference Machines

|      BOARD      |    $MACHINE    | ARCHITECHTURE |
|:---------------:|:--------------:|:-------------:|
|       QEMU      |   qemu-x86-64  |      x86      |
|                 |    qemu-arm    |     arm32     |
|                 |   qemu-arm64   |     arm64     |
|                 |                |               |
|    RCar Gen 3   |     h3ulcb     |     arm64     |
|                 | h3-salvator-x  |     arm64     |
|                 |      h3-kf     |     arm64     |
|                 |     m3ulcb     |     arm64     |
|                 | m3-salvator-x  |     arm64     |
|                 |      m3-kf     |     arm64     |
|                 |                |               |
|  Raspberry Pi   |  raspberrypi4  |     arm64     |

### Community supported Machines

|    BOARD   	|     $MACHINE     	| ARCHITECHTURE |
|:-------------:|:-----------------:|:-------------:|
|  BeagleBone 	|        bbe       	|     arm32     |
|            	|    beaglebone    	|     arm32     |
|            	|                  	|               |
|   i. MX 6  	|      cubox-i     	|     arm32     |
|            	| imx6qdlsabreauto 	|     arm32     |
|            	|    nitrogen6x    	|     arm32     |
|            	|                  	|               |
|   i. MX 8  	|     imx8mqevk    	|     arm64     |
|            	|   imx8mqevk-viv  	|     arm64     |
|            	|                  	|               |
|  Snapdragon 	| dragonboard-410c 	|     arm64     |
|            	| dragonboard-820c 	|     arm64     |
|            	|                  	|               |
|    ARC HS   	|       hsdk       	|      ARC      |


### Supported Images

AGL supports a variety of interfaces, each requiring unique setup configuration.

#### 1. In-Vehicle Infotainment (IVI)

**Supported boards** :

AGL Reference Boards (QEMU, RCar Gen 3 & Raspberry Pi 4)

Community supported Machines (i. MX 6, i. MX 8, Snapdragon & ARC HS)

* Qt Based :

    * Setting up flags at `aglsetup` script :

        ```sh
        $ source meta-agl/scripts/aglsetup.sh -f -m $MACHINE -b build-$MACHINE agl-demo

        #To enable Developer Options
        $ source meta-agl/scripts/aglsetup.sh -f -m $MACHINE -b build-$MACHINE agl-demo agl-devel
        ```

    * Building target image :

        ```sh
        $ time bitbake agl-demo-platform
        ```

* HTML5 Based :

    * Setting up flags at `aglsetup` script :

        ```sh
        $ source meta-agl/scripts/aglsetup.sh -f -m $MACHINE -b build-$MACHINE agl-demo agl-profile-graphical-html5

        # To enable Developer Options
        $ source meta-agl/scripts/aglsetup.sh -f -m $MACHINE -b build-$MACHINE agl-demo agl-profile-graphical-html5 agl-devel
        ```

    * Building target image :

        ```sh
        $ time bitbake agl-demo-platform-html5
        ```


#### 2. Instrument Cluster

**Supported boards** :

AGL Reference Boards (QEMU, RCar Gen 3 & Raspberry Pi 4)

* Setting up flags at `aglsetup` script :

    ```sh
    $ source meta-agl/scripts/aglsetup.sh -f -m $MACHINE -b build-$MACHINE agl-cluster-demo

    # To enable Developer Options
    $ source meta-agl/scripts/aglsetup.sh -f -m $MACHINE -b build-$MACHINE agl-cluster-demo agl-devel
    ```

* Building target image :

    ```sh
    $ time bitbake agl-cluster-demo
    ```

#### 3. Telematics

Headless demo platform for low-spec boards.

**Supported boards** :

Community supported Machines (BeagleBone)


* Setting up flags at `aglsetup` script :

    ```sh
    $ source meta-agl/scripts/aglsetup.sh -f -m $MACHINE -b build-$MACHINE agl-telematics-demo

    # To enable Developer Options
    $ source meta-agl/scripts/aglsetup.sh -f -m $MACHINE -b build-$MACHINE agl-telematics-demo agl-devel
    ```

* Building target image :

    ```sh
    $ time bitbake agl-telematics-demo
    ```