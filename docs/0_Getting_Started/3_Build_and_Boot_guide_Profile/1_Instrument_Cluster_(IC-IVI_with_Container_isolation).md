# Build and Boot AGL Instrument Cluster demo image (IC-IVI with Container isolation)
## Required Equipments
**1) Tested board:** **[Starter Kit Pro/M3](https://elinux.org/R-Car/Boards/M3SK) + [kingfisher support](https://elinux.org/R-Car/Boards/Kingfisher)**

**2) MicroUSB**

**3) MicroSD card**
## 0. Host PC environemnt
**Build PC**
Ubuntu OS (Tested version 18.04.6 LTS)

## 1. Define Your Top-Level Directory

```bash
export AGL_TOP=$HOME/AGL
mkdir -p $AGL_TOP
```

## 2. Download the repo Tool and Set Permissions

```bash
mkdir -p $HOME/bin
export PATH=$HOME/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > $HOME/bin/repo
chmod a+x $HOME/bin/repo
```

## 3. Download the AGL Source Files
- **AGL Version:** Magic Marlin
```bash
cd $AGL_TOP
mkdir marlin
export AGL_TOP=$HOME/AGL/marlin
cd $AGL_TOP
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
repo init -b marlin -u https://gerrit.automotivelinux.org/gerrit/AGL/AGL-repo
repo sync
```
- Optional: Specify the manifest file(marlin_13.0.0.xml) using -m option

```bash
repo init -b marlin -m marlin_13.0.0.xml -u https://gerrit.automotivelinux.org/gerrit/AGL/AGL-repo
```
- Optional: Specify the master branch
```bash
repo init -u https://gerrit.automotivelinux.org/gerrit/AGL/AGL-repo
```

## 4. Downloading Proprietary Drivers
Downloading Proprietary Drivers from [Renesas-automotive-products](https://www.renesas.com/us/en/products/automotive-products/automotive-system-chips-socs/r-car-h3-m3-documents-software)
- To check the files to download
```bash
grep -rn ZIP_.= $AGL_TOP/meta-agl/meta-agl-bsp/meta-rcar-gen3/scripts/setup_mm_packages.sh
export XDG_DOWNLOAD_DIR=$HOME/Downloads
```
- Download and copy Proprietary Drivers files (Run commands at downloaded files directory)
```bash
cp R-Car_Gen3_Series_Evaluation_Software_Package_* $XDG_DOWNLOAD_DIR/
chmod a+rw $XDG_DOWNLOAD_DIR/*.zip
```
## 5. Define Your Board
- Supporting Starter Kit Pro/M3 + kingfisher Board (For other supported boards, check [Define Your Board](https://docs.automotivelinux.org/en/master/#0_Getting_Started/2_Building_AGL_Image/5_3_RCar_Gen_3/))
```bash
export MACHINE=m3ulcb-kf
```
## 6. Run the aglsetup.sh Script
```bash
cd $AGL_TOP
source meta-agl/scripts/aglsetup.sh -f -m $MACHINE -b $MACHINE agl-lxc
```

## 7. Using BitBake
```bash
bitbake lxc-host-image-demo
```
- The build process puts the resulting image in the Build Directory
($AGL_TOP/m3ulcb-kf/tmp/deploy/images/m3ulcb)
## 8.Boot the Board (Deploying the AGL Demo Image)
- To boot your image on the Renesas board, you need to do three things:

a) Update all [firmware](https://docs.automotivelinux.org/en/marlin/#0_Getting_Started/2_Building_AGL_Image/5_3_RCar_Gen_3/#4-troubleshooting) on R-Car M3 Starter Kit board (Flashing firmware).

b) Prepare the MicroSD card and Flash image to the MicroSD card using [Etcher](https://www.balena.io/etcher/)
 (**image file name:** lxc-host-image-demo-m3ulcb-kf.wic.xz), then insert MicroSD card in the R-Car M3SK.  
c) Boot the board.

1) Use a MicroUSB cable to connect the PC to R-Car Starter Kit Pro (M3ULCB) board CN12 "CPLD/DEBUG"
- Serial settings are 115200 8N1.
- Run any standard terminal emulator program (e.g. minicom).
[Replace **"Device"** with USB tty device name, for example **`/dev/ttyUSB0`**]
```bash
sudo minicom -b 115200 -D "DEVICE"
```

- Power on the board
- Quickly hit any key to get into the U-boot command prompt; you should see the following:
 ```bash
Hit any key to stop autoboot:  0                                                                =>
```
- Booting image command (for details check [How to boot](https://elinux.org/R-Car/AGL#Instrument_Cluster_with_Container_isolation_demo_image))
 ```bash
ext4load 0x48080000 Image
ext4load 0x48000000 /boot/r8a77961-ulcb-kf.dtb 
booti 0x48080000 - 0x48000000 
```

# Run SoC board Screen
A) Connect HDMI panel to M3SK(CN4) for **IVI Container**
![IVI](https://elinux.org/images/9/91/Marlin-lxc-Ivi.JPG)

B) Connect HDMI panel to Kingfisher(CN49)for **Cluster Container**
![IC](https://elinux.org/images/7/76/Marlin-lxc-Cluster.JPG)

<! -- https://elinux.org/images/thumb/7/76/Marlin-lxc-Cluster.JPG/1200px-Marlin-lxc-Cluster.JPG) -->
# Reference webpages 
 1. [eLinux](https://elinux.org/R-Car/AGL)
 1. [Kingfisher Board](https://elinux.org/R-Car/Boards/Kingfisher)
 1. [R-Car M3SK](https://elinux.org/R-Car/Boards/M3SK#Flashing_firmware)
 1. [agl reference machines](https://docs.automotivelinux.org/en/master/#1_hardware_support/overview/)
 1. [AGL Tech Day Presenation](https://static.sched.com/hosted_files/agltechday2022/3b/agl-techday-202204.pdf)
 1. [Build AGL Image](https://docs.automotivelinux.org/en/master/#0_Getting_Started/2_Building_AGL_Image/0_Build_Process/)
 1. [Building for Supported Renesas Boards](https://docs.automotivelinux.org/en/master/#0_Getting_Started/2_Building_AGL_Image/5_3_RCar_Gen_3/)

