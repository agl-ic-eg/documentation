---
title: Supported Boards
---

The following table briefs about the various hardware platforms, supported by AGL :

### AGL Reference Machines

|      BOARD      |    $MACHINE    | ARCHITECHTURE |
|:---------------:|:--------------:|:-------------:|
|       QEMU      |   qemu-x86-64  |      x86      |
|                 |    qemu-arm    |     arm 32    |
|                 |   qemu-arm64   |     arm 64    |
|                 |                |               |
|    RCar Gen 3   |     h3ulcb     |     arm 64    |
|                 | h3-salvator-x  |     arm 64    |
|                 |      h3-kf     |     arm 64    |
|                 |     m3ulcb     |     arm 64    |
|                 | m3-salvator-x  |     arm 64    |
|                 |      m3-kf     |     arm 64    |
|                 |                |               |
|  Raspberry Pi   |  raspberrypi4  |     arm 64    |

### Community supported Machines

|    BOARD   	|     $MACHINE     	| ARCHITECHTURE |
|:-------------:|:-----------------:|:-------------:|
|  BeagleBone 	|        bbe       	|     arm 32    |
|            	|    beaglebone    	|     arm 32    |
|            	|                  	|               |
|   i. MX 6  	|      cubox-i     	|     arm 32    |
|            	| imx6qdlsabreauto 	|     arm 32    |
|            	|    nitrogen6x    	|     arm 32    |
|            	|                  	|               |
|   i. MX 8  	|     imx8mqevk    	|     arm 64    |
|            	|   imx8mqevk-viv  	|     arm 64    |
|            	|                  	|               |
|  Snapdragon 	| dragonboard-410c 	|     arm 64    |
|            	| dragonboard-820c 	|     arm 64    |
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

        #To enable Developer Options
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

    #To enable Developer Options
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

    #To enable Developer Options
    $ source meta-agl/scripts/aglsetup.sh -f -m $MACHINE -b build-$MACHINE agl-telematics-demo agl-devel
    ```

* Building target image :

    ```sh
    $ time bitbake agl-telematics-demo
    ```