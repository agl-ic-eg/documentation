---
title: Setting Up AGL SDK
---

AGL provides a pre-built ready-made Software Development Kit (SDK) to help
quickstart the service and application development process.

1. Download the prebuilt SDK :

      - **x86** : [qemux86-64](https://download.automotivelinux.org/AGL/snapshots/master/latest/qemux86-64/deploy/sdk/poky-agl-glibc-x86_64-agl-demo-platform-crosssdk-corei7-64-qemux86-64-toolchain-10.90.0+snapshot.sh)

      - **ARM 32 bit** : [qemuarm](https://download.automotivelinux.org/AGL/snapshots/master/latest/qemuarm/deploy/sdk/poky-agl-glibc-x86_64-agl-demo-platform-crosssdk-armv7vet2hf-neon-vfpv4-qemuarm-toolchain-10.90.0+snapshot.sh)

      - **AARCH64 - ARM 64bit** : [qemuarm64](https://download.automotivelinux.org/AGL/snapshots/master/latest/qemuarm64/deploy/sdk/poky-agl-glibc-x86_64-agl-demo-platform-crosssdk-aarch64-qemuarm64-toolchain-10.90.0+snapshot.sh)

        *Henceforth,* **qemux86-64** *is used in these guides, unless specified
        otherwise.*

2. Create application developmment directory and copy SDK into them :

    ```sh
    $ mkdir ~/agl-app
    $ cp ~/Downloads/poky-agl-glibc-x86_64-agl-demo-platform-crosssdk-corei7-64-qemux86-64-toolchain-10.90.0+snapshot.sh ~/agl-app/
    $ cd ~/agl-app
    ```

3. Install the downloaded SDK :

    ```sh
    $ chmod 777 poky-agl-glibc-x86_64-agl-demo-platform-crosssdk-corei7-64-qemux86-64-toolchain-10.90.0+snapshot.sh
    $ mkdir agl-sdk/
    $ ./poky-agl-glibc-x86_64-agl-demo-platform-crosssdk-corei7-64-qemux86-64-toolchain-10.90.0+snapshot.sh
    ```
    Select target directory for SDK : `~/agl-app/agl-sdk`

    ```sh
    Automotive Grade Linux SDK installer version 10.90.0+snapshot
    =============================================================
    Enter target directory for SDK (default: /opt/agl-sdk/10.90.0+snapshot-corei7-64): ~/agl-app/agl-sdk
    You are about to install the SDK to "/home/boron/agl-app/agl-sdk". Proceed [Y/n]? Y
    Extracting SDK..........................................................................................................................................done
    Setting it up...done
    SDK has been successfully set up and is ready to be used.
    Each time you wish to use the SDK in a new shell session, you need to source the environment setup script e.g.
    $ . /home/boron/agl-app/agl-sdk/environment-setup-corei7-64-agl-linux
    ```

4. Source the SDK environment setup, each time you wish to use the SDK in a new shell session :

    ```sh
    $ source ~/agl-app/agl-sdk/environment-setup-corei7-64-agl-linux
    ```