---
title: Connectivity
---

This part shows different Connectivity attacks on the car.

Domain                  | Improvement
----------------------- | -----------------
Connectivity-Abstract-1 | Improve abstract.



--------------------------------------------------------------------------------



## Acronyms and Abbreviations

The following table lists the terms utilized within this part of the document.

Acronyms or Abbreviations | Description
------------------------- | ---------------------------------------------------------------------------------
_ARP_                     | **A**ddress **R**esolution **P**rotocol
_BLE_                     | **B**luetooth **L**ow **E**nergy
_CAN_                     | **C**ar **A**rea **N**etwork
_CCMP_                    | **C**ounter-Mode/**C**BC-**M**ac **P**rotocol
_EDGE_                    | **E**nhanced **D**ata **R**ates for **GSM** **E**volution - Evolution of **GPRS**
_GEA_                     | **G**PRS **E**ncryption **A**lgorithm
_GPRS_                    | **G**eneral **P**acket **R**adio **S**ervice (2,5G, 2G+)
_GSM_                     | **G**lobal **S**ystem for **M**obile Communications (2G)
_HSPA_                    | **H**igh **S**peed **P**acket **A**ccess (3G+)
_IMEI_                    | **I**nternational **M**obile **E**quipment **I**dentity
_LIN_                     | **L**ocal **I**nterconnect **N**etwork
_MOST_                    | **M**edia **O**riented **S**ystem **T**ransport
_NFC_                     | **N**ear **F**ield **C**ommunication
_OBD_                     | **O**n-**B**oard **D**iagnostics
_PATS_                    | **P**assive **A**nti-**T**heft **S**ystem
_PKE_                     | **P**assive **K**eyless **E**ntry
_PSK_                     | **P**hase-**S**hift **K**eying
_RDS_                     | **R**adio **D**ata **S**ystem
_RFID_                    | **R**adio **F**requency **I**dentification
_RKE_                     | **R**emote **K**eyless **E**ntry
_SDR_                     | **S**oftware **D**efined **R**adio
_SSP_                     | **S**ecure **S**imple **P**airing
_TKIP_                    | **T**emporal **K**ey **I**ntegrity **P**rotocol
_TPMS_                    | **T**ire **P**ressure **M**onitoring **S**ystem
_UMTS_                    | **U**niversal **M**obile **T**elecommunications **S**ystem (3G)
_USB_                     | **U**niversal **S**erial **B**us
_WEP_                     | **W**ired **E**quivalent **P**rivacy
_WPA_                     | **W**ifi **P**rotected **A**ccess

# Bus

We only speak about the **CAN** bus to take an example, because the different
attacks on bus like _FlewRay_, _ByteFlight_, _Most_ and _Lin_ use retro
engineering and the main argument to improve their security is to encrypt data
packets. We just describe them a bit:

- **CAN**: Controller Area Network, developed in the early 1980s, is an
  event-triggered controller network for serial communication with data rates up
  to one MBit/s. **CAN** messages are classified over their respective
  identifier. **CAN** controller broadcast their messages to all connected nodes
  and all receiving nodes decide independently if they process the message.
- **FlewRay**: Is a deterministic and error-tolerant high-speed bus. With a data
  rate up to 10 MBit/s.
- **ByteFlight**: Is used for safety-critical applications in motor vehicles
  like air-bags. Byteflight runs at 10Mbps over 2 or 3 wires plastic optical
  fibers.
- **Most**: Media Oriented System Transport, is used for transmitting audio,
  video, voice, and control data via fiber optic cables. The speed is, for the
  synchronous way, up to 24 MBit/s and asynchronous way up to 14 MBit/s.
  **MOST** messages include always a clear sender and receiver address.
- **LIN**: Local Interconnect Network, is a single-wire subnet work for
  low-cost, serial communication between smart sensors and actuators with
  typical data rates up to 20 kBit/s. It is intended to be used from the year
  2001 on everywhere in a car, where the bandwidth and versatility of a **CAN**
  network is not required.

On just about every vehicle, **ECU**s (**E**lectronic **C**ontrol **U**nits)
communicate over a CAN bus, which is a two-wire bus using hardware arbitration
for messages sent on the shared medium. This is essentially a *trusted* network
where all traffic is visible to all controllers and any controller can send any
message.

A malicious **ECU** on the CAN bus can easily inject messages destined for any
other device, including things like the instrument cluster and the head unit.
There are common ways for hardware to do USB to CAN and open source software to
send and receive messages. For example, there is a driver included in the Linux
kernel that can be used to send/receive CAN signals. A malicious device on the
CAN bus can cause a great number of harmful things to happen to the system,
including: sending bogus information to other devices, sending unintended
commands to ECUs, causing DOS (Denial Of Service) on the CAN bus, etc.



Domain                             | Tech name | Recommendations
---------------------------------- | --------- | --------------------------------------------------------------------------
Connectivity-BusAndConnector-Bus-1 | CAN       | Implement hardware solution in order to prohibit sending unwanted signals.



See [Security in Automotive Bus
Systems](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.92.728&rep=rep1&type=pdf)
for more information.

# Connectors

For the connectors, we supposed that they were disabled by default. For example,
the **USB** must be disabled to avoid attacks like BadUSB. If not, configure the
Kernel to only enable the minimum require **USB** devices. The connectors used
to diagnose the car like **OBD-II** must be disabled outside garages.



Domain                                    | Tech name | Recommendations
----------------------------------------- | --------- | ----------------------------------------------------------------------
Connectivity-BusAndConnector-Connectors-1 | USB       | Must be disabled. If not, only enable the minimum require USB devices.
Connectivity-BusAndConnector-Connectors-2 | USB       | Confidential data exchanged with the ECU over USB must be secure.
Connectivity-BusAndConnector-Connectors-3 | USB       | USB Boot on a ECU must be disable.
Connectivity-BusAndConnector-Connectors-4 | OBD-II    | Must be disabled outside garages.



# Wireless

In this part, we talk about possible remote attacks on a car, according to the
different areas of possible attacks. For each communication channels, we
describe attacks and how to prevent them with some recommendations. The main
recommendation is to always follow the latest updates of these remote
communication channels.



Domain                  | Object | Recommendations
----------------------- | ------ | ------------------------------------------------------------------
Connectivity-Wireless-1 | Update | Always follow the latest updates of remote communication channels.



We will see the following parts:

- [Wifi](#wifi)

- [Bluetooth](#bluetooth)

- [Cellular](#cellular)

- [Radio](#radio)

- [NFC](#nfc)



Domain                  | Improvement
----------------------- | -------------------------------------------
Connectivity-Wireless-1 | Add communication channels (RFID, ZigBee?).



--------------------------------------------------------------------------------

For existing automotive-specific means, we take examples of existing system
attacks from the _IOActive_ document ([A Survey of Remote Automotive Attack
Surfaces](https://www.ioactive.com/pdfs/IOActive_Remote_Attack_Surfaces.pdf))
and from the ETH document ([Relay Attacks on Passive Keyless Entry and Start
Systems in Modern Cars](https://eprint.iacr.org/2010/332.pdf)).

- [Telematics](https://www.ioactive.com/pdfs/IOActive_Remote_Attack_Surfaces.pdf#%5B%7B%22num%22%3A40%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2C60%2C720%2C0%5D)

- [Passive Anti-Theft System
  (PATS)](https://www.ioactive.com/pdfs/IOActive_Remote_Attack_Surfaces.pdf#%5B%7B%22num%22%3A11%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2C60%2C574%2C0%5D)

- [Tire Pressure Monitoring System
  (TPMS)](https://www.ioactive.com/pdfs/IOActive_Remote_Attack_Surfaces.pdf#%5B%7B%22num%22%3A17%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2C60%2C720%2C0%5D)

- [Remote Keyless Entry/Start
  (RKE)](https://www.ioactive.com/pdfs/IOActive_Remote_Attack_Surfaces.pdf#%5B%7B%22num%22%3A26%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2C60%2C720%2C0%5D)

- [Passive Keyless Entry (PKE)](https://eprint.iacr.org/2010/332.pdf)

--------------------------------------------------------------------------------



## Wifi

### Attacks

We can differentiate existing attacks on wifi in two categories: Those on
**WEP** and those on **WPA**.

- **WEP** attacks:

  - **FMS**: (**F**luhrer, **M**antin and **S**hamir attack) is a "Stream cipher
    attack on the widely used RC4 stream cipher. The attack allows an attacker
    to recover the key in an RC4 encrypted stream from a large number of
    messages in that stream."
  - **KoreK**: "Allows the attacker to reduce the key space".
  - **PTW**: (**P**yshkin **T**ews **W**einmann attack).
  - **Chopchop**: Found by KoreK, "Weakness of the CRC32 checksum and the lack
    of replay protection."
  - **Fragmentation**

- **WPA** attacks:

  - **Beck and Tews**: Exploit weakness in **TKIP**. "Allow the attacker to
    decrypt **ARP** packets and to inject traffic into a network, even allowing
    him to perform a **DoS** or an **ARP** poisoning".
  - [KRACK](https://github.com/kristate/krackinfo): (K)ey (R)einstallation
    (A)tta(ck) ([jira AGL
    SPEC-1017](https://jira.automotivelinux.org/browse/SPEC-1017)).

### Recommendations

- Do not use **WEP**, **PSK** and **TKIP**.

- Use **WPA2** with **CCMP**.

- Should protect data sniffing.



Domain                       | Tech name or object | Recommendations
---------------------------- | ------------------- | -------------------------------------------------------------------------
Connectivity-Wireless-Wifi-1 | WEP, PSK, TKIP      | Disabled
Connectivity-Wireless-Wifi-2 | WPA2 and AES-CCMP   | Used
Connectivity-Wireless-Wifi-3 | WPA2                | Should protect data sniffing.
Connectivity-Wireless-Wifi-4 | PSK                 | Changing regularly the password.
Connectivity-Wireless-Wifi-5 | Device              | Upgraded easily in software or firmware to have the last security update.



See [Wifi attacks WEP WPA](https://matthieu.io/dl/wifi-attacks-wep-wpa.pdf) and
[Breaking wep and wpa (Beck and
Tews)](https://dl.aircrack-ng.org/breakingwepandwpa.pdf) for more information.

--------------------------------------------------------------------------------



## Bluetooth

### Attacks

- **Bluesnarfing** attacks involve an attacker covertly gaining access to your
  Bluetooth-enabled device for the purpose of retrieving information, including
  addresses, calendar information or even the device's **I**nternational
  **M**obile **E**quipment **I**dentity. With the **IMEI**, an attacker could
  route your incoming calls to his cell phone.
- **Bluebugging** is a form of Bluetooth attack often caused by a lack of
  awareness. Similar to bluesnarfing, bluebugging accesses and uses all phone
  features but is limited by the transmitting power of class 2 Bluetooth radios,
  normally capping its range at 10-15 meters.
- **Bluejacking** is the sending of unsolicited messages.
- **BLE**: **B**luetooth **L**ow **E**nergy
  [attacks](https://www.usenix.org/system/files/conference/woot13/woot13-ryan.pdf).
- **DoS**: Drain a device's battery or temporarily paralyze the phone.

### Recommendations

- Not allowing Bluetooth pairing attempts without the driver's first manually
  placing the vehicle in pairing mode.
- Monitoring.
- Use **BLE** with caution.
- For v2.1 and later devices using **S**ecure **S**imple **P**airing (**SSP**),
  avoid using the "Just Works" association model. The device must verify that an
  authenticated link key was generated during pairing.



Domain                            | Tech name     | Recommendations
--------------------------------- | ------------- | ------------------------------------------------------------
Connectivity-Wireless-Bluetooth-1 | BLE           | Use with caution.
Connectivity-Wireless-Bluetooth-2 | Bluetooth     | Monitoring
Connectivity-Wireless-Bluetooth-3 | SSP           | Avoid using the "Just Works" association model.
Connectivity-Wireless-Bluetooth-4 | Visibility    | Configured by default as undiscoverable. Except when needed.
Connectivity-Wireless-Bluetooth-5 | Anti-scanning | Used, inter alia, to slow down brute force attacks.



See [Low energy and the automotive
transformation](http://www.ti.com/lit/wp/sway008/sway008.pdf), [Gattacking
Bluetooth Smart Devices](http://gattack.io/whitepaper.pdf), [Comprehensive
Experimental Analyses of Automotive Attack
Surfaces](http://www.autosec.org/pubs/cars-usenixsec2011.pdf) and [With Low
Energy comes Low
Security](https://www.usenix.org/system/files/conference/woot13/woot13-ryan.pdf)
for more information.

--------------------------------------------------------------------------------



## Cellular

### Attacks

- **IMSI-Catcher**: Is a telephone eavesdropping device used for intercepting
  mobile phone traffic and tracking location data of mobile phone users.
  Essentially a "fake" mobile tower acting between the target mobile phone and
  the service provider's real towers, it is considered a man-in-the-middle
  (**MITM**) attack.

- Lack of mutual authentication (**GPRS**/**EDGE**) and encryption with
  **GEA0**.

- **Fall back** from **UMTS**/**HSPA** to **GPRS**/**EDGE** (Jamming against
  **UMTS**/**HSPA**).

- 4G **DoS** attack.

### Recommendations

- Check antenna legitimacy.



Domain                           | Tech name | Recommendations
-------------------------------- | --------- | --------------------------
Connectivity-Wireless-Cellular-1 | GPRS/EDGE | Avoid
Connectivity-Wireless-Cellular-2 | UMTS/HSPA | Protected against Jamming.



See [A practical attack against GPRS/EDGE/UMTS/HSPA mobile data
communications](https://media.blackhat.com/bh-dc-11/Perez-Pico/BlackHat_DC_2011_Perez-Pico_Mobile_Attacks-wp.pdf)
for more information.

--------------------------------------------------------------------------------

## Radio

### Attacks

- Interception of data with low cost material (**SDR** with hijacked DVB-T/DAB
  for example).

### Recommendations

- Use the **R**adio **D**ata **S**ystem (**RDS**) only to send signals for audio
  output and meta concerning radio.



Domain                        | Tech name | Recommendations
----------------------------- | --------- | --------------------------------------------
Connectivity-Wireless-Radio-1 | RDS       | Only audio output and meta concerning radio.



--------------------------------------------------------------------------------



## NFC

### Attacks

- **MITM**: Relay and replay attack.

### Recommendations

- Should implements protection against relay and replay attacks (Tokens,
  etc...).
- Disable unneeded and unapproved services and profiles.
- NFC should be use encrypted link (secure channel). A standard key agreement
  protocol like Diffie-Hellmann based on RSA or Elliptic Curves could be applied
  to establish a shared secret between two devices.
- Automotive NFC device should be certified by NFC forum entity: The NFC Forum
  Certification Mark shows that products meet global interoperability standards.
- NFC Modified Miller coding is preferred over NFC Manchester coding.



Domain                      | Tech name | Recommendations
--------------------------- | --------- | ------------------------------------------------------
Connectivity-Wireless-NFC-1 | NFC       | Protected against relay and replay attacks.
Connectivity-Wireless-NFC-2 | Device    | Disable unneeded and unapproved services and profiles.



# Cloud

## Download

- **authentication**: Authentication is the security process that validates the
  claimed identity of a device, entity or person, relying on one or more
  characteristics bound to that device, entity or person.

- **Authorization**: Parses the network to allow access to some or all network
  functionality by providing rules and allowing access or denying access based
  on a subscriber's profile and services purchased.



Domain                       | Object         | Recommendations
---------------------------- | -------------- | --------------------------------------
Application-Cloud-Download-1 | authentication | Must implement authentication process.
Application-Cloud-Download-2 | Authorization  | Must implement Authorization process.



--------------------------------------------------------------------------------

## Infrastructure

- **Deep Packet Inspection**: **DPI** provides techniques to analyze the payload
  of each packet, adding an extra layer of security. **DPI** can detect and
  neutralize attacks that would be missed by other security mechanisms.

- A **DoS** protection in order to avoid that the Infrastructure is no more
  accessible for a period of time.

- **Scanning tools** such as **SATS** and **DAST** assessments perform
  vulnerability scans on the source code and data flows on web applications.
  Many of these scanning tools run different security tests that stress
  applications under certain attack scenarios to discover security issues.

- **IDS & IPS**: **IDS** detect and log inappropriate, incorrect, or anomalous
  activity. **IDS** can be located in the telecommunications networks and/or
  within the host server or computer. Telecommunications carriers build
  intrusion detection capability in all network connections to routers and
  servers, as well as offering it as a service to enterprise customers. Once
  **IDS** systems have identified an attack, **IPS** ensures that malicious
  packets are blocked before they cause any harm to backend systems and
  networks. **IDS** typically functions via one or more of three systems:

  1. Pattern matching.
  2. Anomaly detection.
  3. Protocol behavior.





Domain                             | Object        | Recommendations
---------------------------------- | ------------- | ----------------------------------------------------------
Application-Cloud-Infrastructure-1 | Packet        | Should implement a DPI.
Application-Cloud-Infrastructure-2 | DoS           | Must implement a DoS protection.
Application-Cloud-Infrastructure-3 | Test          | Should implement scanning tools like SATS and DAST.
Application-Cloud-Infrastructure-4 | Log           | Should implement security tools (IDS and IPS).
Application-Cloud-Infrastructure-5 | App integrity | Applications must be signed by the code signing authority.



--------------------------------------------------------------------------------

## Transport

For data transport, it is necessary to **encrypt data end-to-end**. To prevent
**MITM** attacks, no third party should be able to interpret transported data.
Another aspect is the data anonymization in order to protect the leakage of
private information on the user or any other third party.

The use of standards such as **IPSec** provides "_private and secure
communications over IP networks, through the use of cryptographic security
services, is a set of protocols using algorithms to transport secure data over
an IP network._". In addition, **IPSec** operates at the network layer of the
**OSI** model, contrary to previous standards that operate at the application
layer. This makes its application independent and means that users do not need
to configure each application to **IPSec** standards.

**IPSec** provides the services below :

- Confidentiality: A service that makes it impossible to interpret data if it is
  not the recipient. It is the encryption function that provides this service by
  transforming intelligible (unencrypted) data into unintelligible (encrypted)
  data.
- Authentication: A service that ensures that a piece of data comes from where
  it is supposed to come from.
- Integrity: A service that consists in ensuring that data has not been tampered
  with accidentally or fraudulently.
- Replay Protection: A service that prevents attacks by re-sending a valid
  intercepted packet to the network for the same authorization. This service is
  provided by the presence of a sequence number.
- Key management: Mechanism for negotiating the length of encryption keys
  between two **IPSec** elements and exchange of these keys.

An additional means of protection would be to do the monitoring between users
and the cloud as a **CASB** will provide.



Domain                        | Object                                    | Recommendations
----------------------------- | ----------------------------------------- | ---------------------------------
Application-Cloud-Transport-1 | Integrity, confidentiality and legitimacy | Should implement IPSec standards.

