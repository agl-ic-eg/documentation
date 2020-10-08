---
title: Building for R Car Gen 3
---

AGL supports building for several automotive
[Renesas](https://www.renesas.com/us/en/solutions/automotive.html) board kits.
Renesas is the number one supplier of vehicle control microcontrollers and
System on a Chip (SoC) products for the automotive industry.

This section provides the build and deploy steps you need to create an
image for the following Renesas platforms:

* [Renesas R-Car Starter Kit Pro Board](https://www.elinux.org/R-Car/Boards/M3SK)
* [Renesas R-Car Starter Kit Premier Board](https://www.elinux.org/R-Car/Boards/H3SK)
* [Renesas Salvator-X Board](https://www.elinux.org/R-Car/Boards/Salvator-X)
* [Renesas Kingfisher Infotainment Board](https://elinux.org/R-Car/Boards/Kingfisher)

**NOTE:** You can find similar information for the Pro and Premier board kits on the
[R-Car/Boards/Yocto-Gen3](https://elinux.org/R-Car/Boards/Yocto-Gen3) page.
The information on this page describes setup and build procedures for both these
Renesas development kits.

## 1. Downloading Proprietary Drivers

Before setting up the build environment, you need to download proprietary drivers from the
[R-Car H3/M3 Software library and Technical document](https://www.renesas.com/us/en/solutions/automotive/rcar-download/rcar-demoboard-2.html)
site.

Follow these steps to download the drivers you need:

1. **Determine the Files You Need:**

     Run the `setup_mm_packages.sh` script as follows to
     display the list of ZIP files containing the drivers you need.

     ```sh
     $ grep -rn ZIP_.= $AGL_TOP/meta-agl/meta-agl-bsp/meta-rcar-gen3/scripts/setup_mm_packages.sh
     ```

     The script's output identifies the files you need to download from the page.

2. **Get Your Board Support Package (BSP) Version:**

    Be sure to have the correct BSP version of the R-Car Starter Kit based on the version of the AGL software you are using. Find the appropriate download links on the [R-Car H3/M3 Software library and Technical document](https://www.renesas.com/us/en/solutions/automotive/rcar-download/rcar-demoboard-2.html)
    site. The file pairs are grouped according to the Yocto Project version you are
    using with the AGL software.

        | AGL Version| Renesas Version |
        | AGL master | 3.21.0          |

3. **Download the Files:**

    Start the download process by clicking the download link.
    If you do not have an account with Renesas, you will be asked to register a free account.
    You must register and follow the "Click Through" licensing process
    in order to download these proprietary files.

    If needed, follow the instructions to create the free account by providing the required
    account information.
    Once the account is registered and you are logged in, you can download the files.

    **NOTE:**
    You might have to re-access the
    [original page](https://www.renesas.com/us/en/solutions/automotive/rcar-download/rcar-demoboard-2.html)
    that contains the download links you need after creating the account and logging in.

4. **Create an Environment Variable to Point to Your Download Area:**

    Create and export an environment variable named `XDG_DOWNLOAD_DIR` that points to
    your download directory.

    ```sh
    $ export XDG_DOWNLOAD_DIR=$HOME/Downloads
    ```

5. **Be Sure the Files Have Rights:**

    Be sure you have the necessary rights for the files you downloaded.

    ```sh
    $ chmod a+r $XDG_DOWNLOAD_DIR/*.zip
    ```

## 2. Getting Your Hardware Together

   Gather together this list of hardware items, which is not exhaustive.
   Having these items ahead of time saves you from having to try and
   collect hardware during development:

   Supported Starter

   * Kit Gen3 board with its 5V power supply.
   * Micro USB-A cable for serial console.
      This cable is optional if you are using Ethernet and an SSH connection.
   * USB 2.0 Hub.  The hub is optional but makes it easy to connect multiple USB devices.
   * Ethernet cable.  The cable is optional if you are using a serial console.
   * HDMI type D (Micro connector) cable and an associated display.
   * 4 Gbyte minimum MicroSD Card.  It is recommended that you use a class 10 type.
   * USB touch screen device such as the GeChic 1502i/1503i.  A touch screen device is optional.

  **NOTE:** The Salvator-X Board has NDA restrictions.
  Consequently, less documentation is available for this board both here and across the Internet.

## 3. Making Sure Your Build Environment is Correct

   The
   "[Initializing Your Build Environment](./3_Initializing_Your_Build_Environment.md)"
   section presented generic information for setting up your build environment
   using the `aglsetup.sh` script.
   If you are building an image for a supported Renesas board,
   you need to take steps to make sure your build host is set up correctly.

  1. **Define Your Board:**

    Depending on your Renesas board, define and export a `MACHINE` variable as follows:


    | BOARD  | `MACHINE` |
    |:-:|:-:|
    | Renesas RCar H3  | `MACHINE`= h3ulcb |
    | Renesas RCar H3 w Kingfisher Board  | `MACHINE`= h3ulcb-kf |
    | Renesas RCar H3 w/o gfx blobs  | `MACHINE`= h3ulcb-nogfx |
    | Renesas RCar Salvator/H3  | `MACHINE`= h3-salvator-x |
    | Renesas RCar M3  | `MACHINE`= m3ulcb |
    | Renesas RCar M3 w Kingfisher Board  | `MACHINE`= m3ulcb-kf |
    | Renesas RCar M3 w/o gfx blobs  | `MACHINE`= m3ulcb-nogfx |
    | Renesas RCar Salvator/M3  | `MACHINE`= m3-salvator-x |


    For example, the following command defines and exports the `MACHINE` variable
    for the Starter Kit Pro/H3 Board:

    ```sh
    $ export MACHINE=h3ulcb
    ```

  2. **Run the `aglsetup.sh` Script:**

    Use the following commands to run the AGL Setup script:

    ```sh
    $ cd $AGL_TOP
    $ source meta-agl/scripts/aglsetup.sh -m $MACHINE -b build-$MACHINE agl-devel agl-demo
    which expands to :
    $ source meta-agl/scripts/aglsetup.sh -m h3ulcb -b build-h3ulcb agl-devel agl-demo
    ```

  3. **Examine the Script's Log:**

    Running the `aglsetup.sh` script creates the `setup.log` file, which is in
    the `build-h3ulcb/conf` folder.
    You can examine this log to see the results of the script.
    For example, suppose the graphics drivers were missing or could not be extracted
    when you ran the script.

## 4. Using BitBake

   Start the build using the `bitbake` command.

   **NOTE:** An initial build can take many hours depending on your
   CPU and and Internet connection speeds.
   The build also takes approximately 100G-bytes of free disk space.

   For this example, the target is "agl-demo-platform":

  ``` sh
  $ time bitbake agl-demo-platform
  ```

   The build process puts the resulting image in the Build Directory:
  ``` sh
  build-h3ulcb/tmp/deploy/images/$MACHINE
  ```

## 5. Booting the Image Using a MicroSD Card

  To boot your image on the Renesas board, you need to do three things:

  1. Update all firmware on the board.
  2. Prepare the MicroSD card to you can boot from it.
  3. Boot the board.

  **NOTE:** For subsequent builds, you only have to re-write the MicroSD
  card with a new image.

  * **Updating the Board's Firmware**

    Follow these steps to update the firmware:

    1. **Update the Sample Loader and MiniMonitor:**

        You only need to make these updates one time per device.

        Follow the procedure found on the
        eLinux.org wiki to update to at least version 3.02,
        which is mandatory to run the AGL image ([R-car loader update](https://elinux.org/R-Car/Boards/Kingfisher#How_to_update_of_Sample_Loader_and_MiniMonitor)).

    2. **Update the Firmware Stack:**

        You only need to update the firmware stack if you are
        using the Eel or later (5.0) version of AGL software.

        M3 and H3 Renesas board are AArch64 platforms.
        As such, they have a firmware stack that is divided across: **ARM Trusted Firmware**, **OP-Tee** and **U-Boot**.

        If you are using the Eel (5.0) version or later of the AGL software, you must update
        the firmware using the **[R-car h3ulcb firmware update](http://elinux.org/R-Car/Boards/H3SK#Flashing_firmware)**
        or **[R-car m3ulcb firmware update](https://elinux.org/R-Car/Boards/M3SK#Flashing_firmware)** links from the
        [Embedded Linux Wiki](https://www.elinux.org/Main_Page) (i.e. `elinux.org`).

        The table in the wiki lists the files you need to flash the firmware.
        You can find these files in the following directory:

        ```sh
        $AGL_TOP/build/tmp/deploy/images/$MACHINE
        ```

        **NOTE:** The Salvator-X firmware update process is not documented on eLinux.

  * **Preparing the MicroSD Card**

     ```sh
     $ lsblk
     $ sudo umount <boot_device_name>
     $ xzcat agl-image-ivi-crosssdk-h3ulcb.wic.xz | sudo dd of=<boot_device_name> bs=4M
     $ sync
     ```

  * **Booting the Board**

    Follow these steps to boot the board:

    1. Use the board's power switch to turn off the board.

    2. Insert the MicroSD card into the board.

    3. Verify that you have plugged in the following:

      * An external monitor into the board's HDMI port

      * An input device (e.g. keyboard, mouse, touchscreen, and so forth) into the board's USB ports.

    4. Use the board's power switch to turn on the board.

    After a few seconds, you will see the AGL splash screen on the display and you
    will be able to log in at the console's terminal or using the graphic screen.

## 6. Setting Up the Serial Console

  Setting up the Serial Console involves the following:

  * Installing a serial client on your build host
  * Connecting your build host to your Renesas board's serial port
  * Powering on the board to get a shell at the console
  * Configuring U-Boot parameters
  * Logging into the console
  * Determining the board's IP address

 Brief about each process :

  1. Installing a Serial Client on Your Build Host

    You need to install a serial client on your build host.
    Some examples are [GNU Screen](https://en.wikipedia.org/wiki/GNU_Screen), [picocom](https://linux.die.net/man/8/picocom), and [Minicom](https://en.wikipedia.org/wiki/Minicom). Of these three, "picocom" has less dependencies and is therefore considered the "lightest" solution.

  2. Connecting Your Build Host to Your Renesas Board's Serial Port

    You need to physically connect your build host to the Renesas board using
    a USB cable from the host to the serial CP2102 USP port (i.e. Micro USB-A port)
    on the Renesas board.

    Once you connect the board, determine the device created for the serial link.
    Use the `dmesg` command on your build host.

    ```sh
    dmesg | tail 9
    [2097783.287091] usb 2-1.5.3: new full-speed USB device number 24 using ehci-pci
    [2097783.385857] usb 2-1.5.3: New USB device found, idVendor=0403, idProduct=6001
    [2097783.385862] usb 2-1.5.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    [2097783.385864] usb 2-1.5.3: Product: FT232R USB UART
    [2097783.385866] usb 2-1.5.3: Manufacturer: FTDI
    [2097783.385867] usb 2-1.5.3: SerialNumber: AK04WWCE
    [2097783.388288] ftdi_sio 2-1.5.3:1.0: FTDI USB Serial Device converter detected
    [2097783.388330] usb 2-1.5.3: Detected FT232RL
    [2097783.388658] usb 2-1.5.3: FTDI USB Serial Device converter now attached to ttyUSB0
    ```

    The device created is usually `/dev/ttyUSB0`.
    However, the number might vary depending on other USB serial ports connected to the host.

    To use the link, you need to launch the client.
    Here are three commands, which vary based on the serial client, that show
    how to launch the client:

    ```sh
    picocom -b 115200 /dev/ttyUSB0
    ```

    or

    ```sh
    minicom -b 115200 -D /dev/ttyUSB0
    ```

    or

    ```sh
    screen /dev/ttyUSB0 115200
    ```


  3. Powering on the Board to Get a Shell at the Console

    Both the Pro and Premier kits (e.g.
    [m3ulcb](https://elinux.org/R-Car/Boards/M3SK) and
    [h3ulcb](https://elinux.org/R-Car/Boards/H3SK#Hardware)) have nine
    switches (SW1 through SW9).
    To power on the board, "short-press" SW8, which is the power switch.

    Following, is console output for the power on process for each kit:

    h3ulcb:

    ```sh
    NOTICE:  BL2: R-Car Gen3 Initial Program Loader(CA57) Rev.1.0.7
    NOTICE:  BL2: PRR is R-Car H3 ES1.1
    NOTICE:  BL2: LCM state is CM
    NOTICE:  BL2: DDR1600(rev.0.15)
    NOTICE:  BL2: DRAM Split is 4ch
    NOTICE:  BL2: QoS is Gfx Oriented(rev.0.30)
    NOTICE:  BL2: AVS setting succeeded. DVFS_SetVID=0x52
    NOTICE:  BL2: Lossy Decomp areas
    NOTICE:       Entry 0: DCMPAREACRAx:0x80000540 DCMPAREACRBx:0x570
    NOTICE:       Entry 1: DCMPAREACRAx:0x40000000 DCMPAREACRBx:0x0
    NOTICE:       Entry 2: DCMPAREACRAx:0x20000000 DCMPAREACRBx:0x0
    NOTICE:  BL2: v1.1(release):41099f4
    NOTICE:  BL2: Built : 19:20:52, Jun  9 2016
    NOTICE:  BL2: Normal boot
    NOTICE:  BL2: dst=0xe63150c8 src=0x8180000 len=36(0x24)
    NOTICE:  BL2: dst=0x43f00000 src=0x8180400 len=3072(0xc00)
    NOTICE:  BL2: dst=0x44000000 src=0x81c0000 len=65536(0x10000)
    NOTICE:  BL2: dst=0x44100000 src=0x8200000 len=524288(0x80000)
    NOTICE:  BL2: dst=0x49000000 src=0x8640000 len=1048576(0x100000)


    U-Boot 2015.04 (Jun 09 2016 - 19:21:52)

    CPU: Renesas Electronics R8A7795 rev 1.1
    Board: H3ULCB
    I2C:   ready
    DRAM:  3.9 GiB
    MMC:   sh-sdhi: 0, sh-sdhi: 1
    In:    serial
    Out:   serial
    Err:   serial
    Net:   Board Net Initialization Failed
    No ethernet found.
    Hit any key to stop autoboot:  0
    ```


    m3ulcb:

    ```sh
    NOTICE:  BL2: R-Car Gen3 Initial Program Loader(CA57) Rev.1.0.14
    NOTICE:  BL2: PRR is R-Car M3 Ver1.0
    NOTICE:  BL2: Board is Starter Kit Rev1.0
    NOTICE:  BL2: Boot device is HyperFlash(80MHz)
    NOTICE:  BL2: LCM state is CM
    NOTICE:  BL2: AVS setting succeeded. DVFS_SetVID=0x52
    NOTICE:  BL2: DDR1600(rev.0.22)NOTICE:  [COLD_BOOT]NOTICE:  ..0
    NOTICE:  BL2: DRAM Split is 2ch
    NOTICE:  BL2: QoS is default setting(rev.0.17)
    NOTICE:  BL2: Lossy Decomp areas
    NOTICE:       Entry 0: DCMPAREACRAx:0x80000540 DCMPAREACRBx:0x570
    NOTICE:       Entry 1: DCMPAREACRAx:0x40000000 DCMPAREACRBx:0x0
    NOTICE:       Entry 2: DCMPAREACRAx:0x20000000 DCMPAREACRBx:0x0
    NOTICE:  BL2: v1.3(release):4eef9a2
    NOTICE:  BL2: Built : 00:25:19, Aug 25 2017
    NOTICE:  BL2: Normal boot
    NOTICE:  BL2: dst=0xe631e188 src=0x8180000 len=512(0x200)
    NOTICE:  BL2: dst=0x43f00000 src=0x8180400 len=6144(0x1800)
    NOTICE:  BL2: dst=0x44000000 src=0x81c0000 len=65536(0x10000)
    NOTICE:  BL2: dst=0x44100000 src=0x8200000 len=524288(0x80000)
    NOTICE:  BL2: dst=0x50000000 src=0x8640000 len=1048576(0x100000)


    U-Boot 2015.04-dirty (Aug 25 2017 - 10:55:49)

    CPU: Renesas Electronics R8A7796 rev 1.0
    Board: M3ULCB
    I2C:   ready
    DRAM:  1.9 GiB
    MMC:   sh-sdhi: 0, sh-sdhi: 1
    In:    serial
    Out:   serial
    Err:   serial
    Net:   ravb
    Hit any key to stop autoboot:  0
    ```

## 7. Setting-up U-boot

  **Configuring U-Boot Parameters**

    Follow these steps to configure the board to use the MicroSD card as the
    boot device and also to set the screen resolution:

  1. As the board is powering up, press any key to stop the autoboot process.
      You need to press a key quickly as you have just a few seconds in which to
      press a key.

  2. Once the autoboot process is interrupted, use the board's serial console to
     enter `printenv` to check if you have correct parameters for booting your board:

      Here is an example using the **h3ulcb** board:

      ```sh
      $ printenv
      baudrate=115200
      bootargs=console=ttySC0,115200 root=/dev/mmcblk1p1 rootwait ro rootfstype=ext4
      bootcmd=run load_ker; run load_dtb; booti 0x48080000 - 0x48000000
      bootdelay=3
      fdt_high=0xffffffffffffffff
      initrd_high=0xffffffffffffffff
      load_dtb=ext4load mmc 0:1 0x48000000 /boot/r8a7795-h3ulcb.dtb
      load_ker=ext4load mmc 0:1 0x48080000 /boot/Image
      stderr=serial
      stdin=serial
      stdout=serial
      ver=U-Boot 2015.04 (Jun 09 2016 - 19:21:52)

      Environment size: 648/131068 bytes
      ```

      Here is a second example using the **m3ulcb** board:

      ```sh
      $ printenv
      baudrate=115200
      bootargs=console=ttySC0,115200 root=/dev/mmcblk1p1 rootwait ro rootfstype=ext4
      bootcmd=run load_ker; run load_dtb; booti 0x48080000 - 0x48000000
      bootdelay=3
      fdt_high=0xffffffffffffffff
      filesize=cdeb
      initrd_high=0xffffffffffffffff
      load_dtb=ext4load mmc 0:1 0x48000000 /boot/r8a7796-m3ulcb.dtb
      load_ker=ext4load mmc 0:1 0x48080000 /boot/Image
      stderr=serial
      stdin=serial
      stdout=serial
      ver=U-Boot 2015.04 (Nov 30 2016 - 18:25:18)

      Environment size: 557/131068 bytes
      ```

  3. Loading dtb :

      **NOTE** : Refer [here](https://elinux.org/R-Car/Boards/Yocto-Gen3-CommonFAQ/Which_dtb_file_is_required_to_boot_linux_on_the_R-Car_Starter_Kit_board_%3F) for more information.

      Make sure your ``load_dtb`` is set as follows :

       * **H3SK v2.0(DDR 4GB)** : `$ setenv load_dtb ext4load mmc 0:1 0x48000000 /boot/r8a7795-h3ulcb.dtb`

       * **H3SK v2.0(DDR 8GB)/v3.0(DDR 8GB)** : `$ setenv load_dtb ext4load mmc 0:1 0x48000000 /boot/r8a7795-h3ulcb-4x2g.dtb`

       * **M3SK v1.0** : `$ setenv load_dtb ext4load mmc 0:1 0x48000000 /boot/r8a7796-m3ulcb.dtb`

       * **M3SK v3.0** : `$ setenv load_dtb ext4load mmc 0:1 0x48000000 /boot/r8a7796-m3ulcb-2x4g.dtb`

       * **H3SK with a Kingfisher board** : `$ setenv load_dtb ext4load mmc 0:1 0x48000000 /boot/r8a7795-h3ulcb-kf.dtb`

       * **M3SK with a Kingfisher board** : `$ setenv load_dtb ext4load mmc 0:1 0x48000000 /boot/r8a7796-m3ulcb-kf.dtb`

  4. Set Correct Environment :

     Be sure your environment is set up as follows:

     ```sh
     $ setenv bootargs console=ttySC0,115200 ignore_loglevel vmalloc=384M video=HDMI-A-1:1920x1080-32@60 root=/dev/mmcblk1p1 rw rootfstype=ext4 rootwait rootdelay=2
     $ setenv bootcmd run load_ker\; run load_dtb\; booti 0x48080000 - 0x48000000
     $ setenv load_ker ext4load mmc 0:1 0x48080000 /boot/Image
     ```

  5. Save the boot environment: `$ saveenv`

  6. Boot the board: `$ run bootcmd`

## 8. Troubleshooting

  * **Logging Into the Console**

    Once the board boots, you should see the
    [Wayland display](https://en.wikipedia.org/wiki/Wayland_(display_server_protocol))
    on the external monitor.
    A login prompt should appear as follows depending on your board:

    **h3ulcb**:
    ```sh
    Automotive Grade Linux ${AGL_VERSION} h3ulcb ttySC0

    h3ulcb login: root
    ```

    **m3ulcb**:
    ```sh
    Automotive Grade Linux ${AGL_VERSION} m3ulcb ttySC0

    m3ulcb login: root
    ```

    At the prompt, login by using `root` as the login.
    The password is "empty" so you should not be prompted for the password.

  * **Determining the Board's IP Address**

    If your board is connected to a local network using Ethernet and
    if a DHCP server is able to distribute IP addresses,
    you can determine the board's IP address and log in using `ssh`.

    Here is an example for the m3ulcb board:

    ```sh
    m3ulcb login: root
    root@m3ulcb:~# ip -4 a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
        inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
    3: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        inet 10.0.0.27/24 brd 10.0.0.255 scope global eth0
          valid_lft forever preferred_lft forever
    ```

    In the previous example, IP address is 10.0.0.27.
    Once you know the address, you can use `ssh` to login.

    ```sh
    $ ssh root@10.0.0.27
    ```

## 9. Supplementary Information

  * R-Car Generation 3 Information

    Refer to the following for more information from [eLinux website](https://elinux.org/R-Car).

  * Proprietary libraries for meta-rcar-gen3

    The meta-rcar-gen3 layer of meta-renesas is supported Graphic GLES(GSX)
    libraries, proprietary library of multimedia, and ICCOM software.

    1. Build with Renesas multimedia libraries

        Multimedia portions depend on GLES portions.

        * A. Configuration for Multimedia features


            * Please copy proprietary libraries to the directory of recipes.

            * Please set local.conf the following.
              **Enable multimedia features. This provides package group of plug-ins of the GStreamer, multimedia libraries and kernel drivers.**

              ```sh
              MACHINE_FEATURES_append = " multimedia"
              ```

        * B. Configuration for optional codecs and middleware

            * Please copy proprietary libraries to the directory of recipes.

            * Add features to DISTRO_FEATURES_append to local.conf

              **Additional configuration in OMX module**

              ```sh
              " h263dec_lib"       - for OMX Media Component H263 Decoder Library
              " h264dec_lib"       - for OMX Media Component H264 Decoder Library
              " h264enc_lib"       - for OMX Media Component H.264 Encoder Library
              " h265dec_lib"       - for OMX Media Component H265 Decoder Library
              " mpeg2dec_lib"      - for OMX Media Component MPEG2 Decoder Library
              " mpeg4dec_lib"      - for OMX Media Component MPEG4 Decoder Library
              " vc1dec_lib"        - for OMX Media Component VC-1 Decoder Library
              " divxdec_lib"       - for OMX Media Component DivX Decoder Library
              " rvdec_lib"         - for OMX Media Component RealVideo Decoder Library
              " alacdec_lib"       - for OMX Media Component ALAC Decoder Library
              " flacdec_lib"       - for OMX Media Component FLAC Decoder Library
              " aaclcdec_lib"      - for OMX Media Component AAC-LC Decoder Library
              " aaclcdec_mdw"      - for AAC-LC 2ch Decoder Middleware for Linux
              " aacpv2dec_lib"     - for OMX Media Component aacPlus V2 Decoder Library
              " aacpv2dec_mdw"     - for aacPlus V2 Decoder Middleware for Linux
              " mp3dec_lib"        - for OMX Media Component MP3 Decoder Library
              " mp3dec_mdw"        - for MP3 Decoder Middleware for Linux
              " wmadec_lib"        - for OMX Media Component WMA Standard Decoder Library
              " wmadec_mdw"        - for WMA Standard Decoder Middleware for Linux
              " dddec_lib"         - for OMX Media Component Dolby(R) Digital Decoder Library
              " dddec_mdw"         - for Dolby(R) Digital Decoder Middleware for Linux
              " aaclcenc_lib"      - for OMX Media Component AAC-LC Encoder Library
              " vp8dec_lib"        - for OMX Media Component VP8 Decoder Library for Linux
              " vp8enc_lib"        - for OMX Media Component VP8 Encoder Library for Linux
              " vp9dec_lib"        - for OMX Media Component VP9 Decoder Library for Linux
              " aaclcenc_mdw"      - for AAC-LC Encoder Middleware for Linux
              " cmsbcm"            - for CMS Basic Color Management Middleware for Linux
              " cmsblc"            - for CMS CMM3 Backlight Control Middleware for Linux
              " cmsdgc"            - for CMS VSP2 Dynamic Gamma Correction Middleware for Linux
              " dtv"               - for ISDB-T DTV Software Package for Linux
              " dvd"               - for DVD Core-Middleware for Linux
              " adsp"              - for ADSP driver, ADSP interface and ADSP framework for Linux
              " avb"               - for AVB Software Package for Linux
              ```

            Ex:

            ```sh
            DISTRO_FEATURES_append = " h264dec_lib h265dec_lib mpeg2dec_lib aaclcdec_lib aaclcdec_mdw"
            ```

        * C. Configuration for test packages

          Must ensure that Multimedia features have been enabled.
          (Please refer to III/A to enable Multimedia.)

          * Please add feature to DISTRO_FEATURES_append to local.conf.

            **Configuration for multimedia test package**
            ```sh
            DISTRO_FEATURES_append = "mm-test"
            ```

    2. Enable Linux ICCOM driver and Linux ICCOM library


        For Linux ICCOM driver and Linux ICCOM library

        * Please copy proprietary libraries to the directory of recipes.

        * Please set local.conf the following.

        ```sh
        DISTRO_FEATURES_append = "iccom"
        ```


