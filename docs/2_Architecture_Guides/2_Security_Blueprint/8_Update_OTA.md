---
title: Update (Over The Air)
---

Updating applications and firmware is essential for the development of new
features and even more to fix security bugs. However, if a malicious third party
manages to alter the content during transport, it could alter the functioning of
the system and/or applications. The security of the updates is therefore a
critical point to evaluate in order to guarantee the integrity, the
confidentiality and the legitimacy of the transmitted data.

## Attack Vectors

Updates Over The Air are one of the most common points where an attacker will
penetrate. An OTA update mechanism is one of the highest threats in the system.
If an attacker is able to install his own application or firmware on the system,
he can get the same level of access that the original application or firmware
had. From that point, the intruder can get unfettered access to the rest of the
system, which might include making modifications, downloading other pieces of
software, and stealing assets.

### Man In The Middle (MITM)

The man-in-the-middle attack is the most classic example of an attack, where an
adversary inserts himself between two communicating entities and grabs whatever
is being communicated. In the case of OTA attacks, the connection in the network
may be intercepted:

* On the internet, before the cloud back-end.
* At the base station, 3G,4G,5G connection to the internet.
* At the receiving antenna, connection to the car.
* Between the receiving antenna and the gateway router (if present), connection
  within the car.
* Between the gateway router and the target component (IVI, In-Vehicle
  Infotainment unit).

There are many ways to mount a MITM attack. For example, proxy tools like Burp
Proxy can be used to intercept web traffic as a man-in-the-middle. Under the
guise of being a testing tool, the proxy server is often used in attack
scenarios. It runs on a variety of platforms.

As another example, false base station attacks are known to be fairly easy to
set-up. The problem is apparently fairly prevalent in countries like China and
in the UK. These fake base stations are sometimes just eavesdropping on the
communication, but others have the potential to do serious harm.

Defenses against MITM attacks include encrypted and signed data pipes.
Furthermore, architects and developers are also recommended to encrypt and sign
the payloads that are being passed over these pipes, to defend against perusal
of the data.

### Man At The End (MATE)

The man-at-the-end attack is when an intruder analyzes the end-point of the
communication when software is accessing the data communication. This is a more
severe attack type where the attacker can:

* Steal keys.
  * For example, a simple debugging session in running software could reveal a
    key used in memory.
* Tamper software.
  * For example, replacing just one function call in software with a NOP (i.e.
    no operation) can drastically change the behavior of the program.
* Jam branches of control.
  * For example, making a program take one branch of control rather than the
    intended branch can mean the difference between an authorized and a
    non-authorized installation.
* Modify important data.
  * For example, if the data changed is a key or data leading to a control path,
    then this attack can be severe.
  * In the case of OTA updates, MATE attacks are particularly problematic for
    the system. One of the consequences of MATE attacks can be installed
    software that allows installation of any other software. For example, an
    attacker might install remote access software to control any part of the
    system.

--------------------------------------------------------------------------------

## Acronyms and Abbreviations

The following table lists the terms utilized within this part of the document.

Acronyms or Abbreviations | Description
------------------------- | -------------------------------------------------------------------------
_FOTA_                    | **F**irmware **O**ver **T**he **A**ir
_MATE_                    | **M**an **A**t **T**he **E**nd
_MITM_                    | **M**an **I**n **T**he **M**iddle
_OTA_                     | **O**ver **T**he **A**ir
_SOTA_                    | **S**oftware **O**ver **T**he **A**ir
_TUF_                     | **T**he **U**pdate **F**ramework

# Firmware Over The Air

The firmware update is critical since its alteration back to compromise the
entire system. It is therefore necessary to take appropriate protective
measures.

AGL includes the _meta-updater_ Yocto layer that enables OTA software updates
via [Uptane](https://uptane.github.io), an automotive-specific extension to [The
Update Framework](https://theupdateframework.github.io/). Uptane and TUF are
open standards that define a secure protocol for delivering and verifying
updates even when the servers and network--internet and car-internal--aren't
fully trusted.

_meta-updater_ includes the application
[`aktualizr`](https://github.com/advancedtelematic/aktualizr), developed
Advanced Telematic Systems (now part of HERE Technologies) that enables OTA for
an ECU. `aktualizr` combined with Uptane is suitable for updating the firmware,
software, and other packages on even functionally critical ECUs. `aktualizr` can
be enabled with the free, open souce backend
[`ota-community-edition`](https://github.com/advancedtelematic/ota-community-edition).

This FOTA update mechanism can be enabled through the `agl-sota` feature.

## Building

To build an AGL image that uses `aktualizr`, the following can be used.

```
source meta-agl/scripts/aglsetup.sh -m <machine> agl-sota <other-features...>
```

During the build, _meta-updater_ will use credentials downloaded from
`ota-community-edition` to sign metadata verifying the build as authentic. These
signatures are part of the Uptane framework and are used to verify FOTA updates.

## Atomic Upgrades with Rollbacks

`aktualizr`'s primary method of updating firmware is to use `libostree` with
binary diffs. The binary diffs use the least amout of bandwidth, and by it's
nature `libostree` stores current and previous firmware versions on disk or in
flash memory to allow for rollbacks.

`libostree` is a content addressable object store much like `git`. Versions are
specified via SHA2-256. These hashes are signed in the Uptane metadata and are
robust against cryptographic compromise.

# Software Over The Air

Software updates in connected vehicles are a very useful feature, which can
deliver significant benefits. If not implemented with security in mind, software
updates can incur serious vulnerabilities. Any software update system must
ensure that not only are the software updates to devices done in a secure way,
but also that the repositories and servers hosting these updates are adequately
protected. As the process of updating software migrates from a Dealership update
model towards an **OTA** update model, securing these processes becomes a high
priority.

**SOTA** is made possible by **AppFw** (See Platform part). It will be possible
to manage in a simple way the packets (i.g. Android like).

Domain        | Improvement
------------- | -----------------
Update-SOTA-1 | Part to complete.