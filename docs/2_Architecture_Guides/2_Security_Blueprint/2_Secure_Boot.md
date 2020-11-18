---
title: Secure Boot
---

Domain          | Improvement
--------------- | ----------------------------------------------------
Boot-Abstract-1 | More generic and add examples (The chain of trust).

Secure boot refers to preventing malicious software applications and
“unauthorized” operating systems from loading during the system start-up
process. The goal is to protect users from rootkits and other low-level malware
attacks. Modern bootloaders come with features that can be used to enable secure
boot in the system.

**Boot Hardening**: Steps/requirements to configure the boot sequence, in order
to restrict the device from executing anything other than the approved software
image.

In this part, we will see a series of settings that will allow us to improve
security during boot phase. For the purposes of reference and explanation, we
are providing guidance on how to configure an embedded device that runs with a
3.10.17 Linux kernel. If the integrity is not checked or if a critical error
occurs, the system must boot on a very stable backup image.

**Requirements**: These requirements must be met even if an alternative version
of the Linux kernel is chosen.

**Recommendations**: Detailed best practices that should be applied in order to
secure a device. Although they are not currently listed as hard requirements,
they may be upgraded to requirements status in the future. In addition, specific
operators may change some of these recommendations into requirements based on
their specific needs and objectives.

Domain          | Improvement
--------------- | -------------------------------------------
Boot-Abstract-1 | Review the definition of the "boot loader".

**Boot loader**: The boot loader consists of the Primary boot loader residing in
**OTP** memory, sboot, U-Boot and Secure loader residing in external flash (NAND
or SPI/NOR flash memory). The CPU on power on or reset executes the primary boot
loader. The **OTP** primary boot loader makes the necessary initial system
configuration and then loads the secondary boot loader sboot from external flash
memory to ram memory. The sboot then loads the U-Boot along with the Secure
loader. U-Boot then verifies the Kernel/system image integrity, then loads the
Kernel/system image before passing control to it.

--------------------------------------------------------------------------------

## Acronyms and Abbreviations

The following table lists the terms utilized within this part of the document.

Acronyms or Abbreviations | Description
------------------------- | -----------------------------------------------------------------------
_FUSE_                    | **F**ilesystem in **U**ser**S**pac**E**
_OTP_                     | **O**ne-**T**ime-**P**rogrammable
_DOCSIS_                  | **D**ata **O**ver **C**able **S**ervice **I**nterface **S**pecification

# Image

## Image selection

The boot process shall be uninterruptible and shall irrevocably boot the image
as specified in the boot environment.

In U-Boot set the "_bootdelay_" environment variable and/or define
`CONFIG_BOOTDELAY` to _-2_.

Domain                 | _Variable_ / `Config` name | `Value`
---------------------- | -------------------------- | -------
Boot-Image-Selection-1 | `CONFIG_BOOTDELAY`         | `-2`
Boot-Image-Selection-2 | _bootdelay_                | `-2`

--------------------------------------------------------------------------------

## Image authenticity

It shall not be possible to boot from an unverified image. The secure boot
feature in U-Boot shall be enabled. The secure boot feature is available from
U-Boot 2013.07 version. To enable the secure boot feature, enable the following
features:

```sh
CONFIG_FIT: Enables support for Flat Image Tree (FIT) uImage format.
CONFIG_FIT_SIGNATURE: Enables signature verification of FIT images.
CONFIG_RSA: Enables RSA algorithm used for FIT image verification.
CONFIG_OF_CONTROL: Enables Flattened Device Tree (FDT) configuration.
CONFIG_OF_SEPARATE: Enables separate build of u-Boot from the device tree.
CONFIG_DEFAULT_DEVICE_TREE: Specifies the default Device Tree used for the run-time configuration of U-Boot.
```

Generate the U-Boot image with public keys to validate and load the image. It
shall use RSA2048 and SHA256 for authentication.

Domain                    | `Config` name                | _State_
------------------------- | ---------------------------- | --------
Boot-Image-Authenticity-1 | `CONFIG_FIT`                 | _Enable_
Boot-Image-Authenticity-2 | `CONFIG_FIT_SIGNATURE`       | _Enable_
Boot-Image-Authenticity-3 | `CONFIG_RSA`                 | _Enable_
Boot-Image-Authenticity-4 | `CONFIG_OF_CONTROL`          | _Enable_
Boot-Image-Authenticity-5 | `CONFIG_OF_SEPARATE`         | _Enable_
Boot-Image-Authenticity-6 | `CONFIG_DEFAULT_DEVICE_TREE` | _Enable_

# Communication modes

## Disable USB, Serial and DOCSIS Support

To disable USB support in U-Boot, following config's shall not be defined:

```
CONFIG_CMD_USB: Enables basic USB support and the usb command.
CONFIG_USB_UHCI: Defines the lowlevel part.
CONFIG_USB_KEYBOARD: Enables the USB Keyboard.
CONFIG_USB_STORAGE: Enables the USB storage devices.
CONFIG_USB_HOST_ETHER: Enables USB Ethernet adapter support.
```

In addition, disable unnecessary communication modes like Ethernet, Serial
ports, DOCSIS in U-Boot and sboot that are not necessary.

Linux Kernel support for USB should be compiled-out if not required. If it is
needed, the Linux Kernel should be configured to only enable the minimum
required USB devices. User-initiated USB-filesystems should be treated with
special care. Whether or not the filesystems are mounted in userspace
(**FUSE**), restricted mount options should be observed.

Domain               | Communication modes       | _State_
-------------------- | ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------
Boot-Communication-1 | `USB`                     | _Disabled_ and _Compiled-out_ if not required.
Boot-Communication-2 | `USB`                     | Else, Kernel should be configured to only enable the minimum required USB devices and filesystems should be treated with special care.
Boot-Communication-3 | `Ethernet`                | _Disabled_
Boot-Communication-4 | U-boot and sboot `DOCSIS` | _Disabled_
Boot-Communication-5 | `Serial ports`            | _Disabled_

Domain                   | `Config` name           | _State_
------------------------ | ----------------------- | -------------
Boot-Communication-USB-1 | `CONFIG_CMD_USB`        | _Not defined_
Boot-Communication-USB-2 | `CONFIG_USB_UHCI`       | _Not defined_
Boot-Communication-USB-3 | `CONFIG_USB_KEYBOARD`   | _Not defined_
Boot-Communication-USB-4 | `CONFIG_USB_STORAGE`    | _Not defined_
Boot-Communication-USB-5 | `CONFIG_USB_HOST_ETHER` | _Not defined_

--------------------------------------------------------------------------------

## Disable all unused Network Interfaces

Only used network interfaces should be enabled. Where possible, services should
also be limited to those necessary.

Domain               | Communication modes  | _State_
-------------------- | -------------------- | ---------------------------------------------------------------------------------------------
Boot-Communication-1 | `Network interfaces` | Preferably _no network interface is allowed_, otherwise, restrict the services to those used.

## Remove or Disable Unnecessary Services, Ports, and Devices

Restrict the `services`, `ports` and `devices` to those used.

Domain               | Object                            | Recommendations
-------------------- | --------------------------------- | -------------------------------------------------------------
Boot-Communication-1 | `Services`, `ports` and `devices` | Restrict the `services`, `ports` and `devices` to those used.

## Disable flash access

**Recommendation**:

In U-Boot following flash memory commands shall be disabled:

**NAND**: Support for nand flash access available through `do_nand` has to be
disabled.

Domain                     | `Command` name | _State_
-------------------------- | -------------- | ---------
Boot-Communication-Flash-1 | `do_nand`      | _Disable_

Similarly sboot should disable flash access support through command line if any.

# Consoles

## Disable serial console

Serial console output shall be disabled. To disable console output in U-Boot,
set the following macros:

Domain                 | `Config` name                           | `Value`
---------------------- | --------------------------------------- | ---------
Boot-Consoles-Serial-1 | `CONFIG_SILENT_CONSOLE`                 | `Disable`
Boot-Consoles-Serial-2 | `CONFIG_SYS_DEVICE_NULLDEV`             | `Disable`
Boot-Consoles-Serial-3 | `CONFIG_SILENT_CONSOLE_UPDATE_ON_RELOC` | `Disable`

Domain          | Improvement
--------------- | ------------------------------------
Boot-Consoles-1 | Secure loader: No reference earlier?

And set "**silent**" environment variable. For the Secure loader, disable the
traces by not defining the below macro:

Domain                 | `Environment variable` name | _State_
---------------------- | --------------------------- | -------------
Boot-Consoles-Serial-1 | `INC_DEBUG_PRINT`           | _Not defined_

For sboot proper configuration needs to be done to disable the serial console.

--------------------------------------------------------------------------------

## Immutable environment variables

In U-Boot, ensure Kernel command line, boot commands, boot delay and other
environment variables are immutable. This will prevent side-loading of alternate
images, by restricting the boot selection to only the image in FLASH.

The environment variables shall be part of the text region in U-Boot as default
environment variable and not in non-volatile memory.

Remove configuration options related to non-volatile memory, such as:

Domain                     | `Config` name                | _State_
-------------------------- | ---------------------------- | ---------
Boot-Consoles-Variables-1  | `CONFIG_ENV_IS_IN_MMC`       | `#undef`
Boot-Consoles-Variables-2  | `CONFIG_ENV_IS_IN_EEPROM`    | `#undef`
Boot-Consoles-Variables-3  | `CONFIG_ENV_IS_IN_FLASH`     | `#undef`
Boot-Consoles-Variables-4  | `CONFIG_ENV_IS_IN_DATAFLASH` | `#undef`
Boot-Consoles-Variables-5  | `CONFIG_ENV_IS_IN_FAT`       | `#undef`
Boot-Consoles-Variables-6  | `CONFIG_ENV_IS_IN_NAND`      | `#undef`
Boot-Consoles-Variables-7  | `CONFIG_ENV_IS_IN_NVRAM`     | `#undef`
Boot-Consoles-Variables-8  | `CONFIG_ENV_IS_IN_ONENAND`   | `#undef`
Boot-Consoles-Variables-9  | `CONFIG_ENV_IS_IN_SPI_FLASH` | `#undef`
Boot-Consoles-Variables-10 | `CONFIG_ENV_IS_IN_REMOTE`    | `#undef`
Boot-Consoles-Variables-11 | `CONFIG_ENV_IS_IN_UBI`       | `#undef`
Boot-Consoles-Variables-12 | `CONFIG_ENV_IS_NOWHERE`      | `#define`

--------------------------------------------------------------------------------

## (Recommendation) Removal of memory dump commands

In U-Boot, following commands shall be disabled to avoid memory dumps:

```
md : Memory Display command.
mm : Memory modify command - auto incrementing address.
nm : Memory modify command - constant address.
mw : Memory write.
cp : Memory copy.
mwc : Memory write cyclic.
mdc : Memory display cyclic.
mtest : Simple ram read/write test.
loopw : Infinite write loop on address range.
```

Domain                  | `Command` name | _State_
----------------------- | -------------- | ----------
Boot-Consoles-MemDump-1 | `md`           | _Disabled_
Boot-Consoles-MemDump-2 | `mm`           | _Disabled_
Boot-Consoles-MemDump-3 | `nm`           | _Disabled_
Boot-Consoles-MemDump-4 | `mw`           | _Disabled_
Boot-Consoles-MemDump-5 | `cp`           | _Disabled_
Boot-Consoles-MemDump-6 | `mwc`          | _Disabled_
Boot-Consoles-MemDump-7 | `mdc`          | _Disabled_
Boot-Consoles-MemDump-8 | `mtest`        | _Disabled_
Boot-Consoles-MemDump-9 | `loopw`        | _Disabled_

Similarly, memory dump support shall be disabled from sboot.