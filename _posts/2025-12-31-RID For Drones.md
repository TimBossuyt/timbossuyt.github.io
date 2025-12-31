---
title: Direct Remote ID for Drones Using OpenDroneID
date: 2025-12-31 14:00:00 +0100
categories: [Drones]
tags: [drone, remoteid]
description: 
---

## Introduction
As of January 1, 2024, the implementation of Remote Identification for Drones (RID) has become mandatory for Unmanned Aircraft Systems (UAS) operating in both the specific and open categories (except for drones weighing less than 250 grams). This regulation addresses the growing need for drone management and community safety by allowing real-time identification of drones during a flight.

### Drone Categories
A UAS operation is classified as either 'specific' or 'open' based on the associated risk levels. In the specific category, operations are considered higher risk and therefore are subject to stricter regulations e.g. authorization from National Aviation Authority (NAA) or a risk assessment. The open category is intended for lower-risk drone operations and does not require authorization.

### Remote Identification
Remote identification enables authorities, as well as members of the public, to identify drones in real-time using a smartphone app. This system broadcasts details such as the drone's location, the operator's position and registration number. Exposing these details helps to increase public acceptance of drones by promoting accountability and transparency but also aids in faster responses to unauthorized flights.

There are currently two types of RID technology: Direct Remote ID (DRI) and Network Remote ID (NRI). DRI is the most frequently used method because it can use the commonly accessible Wi-Fi and Bluetooth channels to broadcast the data. NRI, on the other hand, is used exclusively for special operations within the specific category and transmits information over an internet connection. The rest of this paper will focus on DRI.
## Regulations in the EU
### Regulation 2019/945
EU Regulation 2019/945 establishes a legal framework for the safe operation and management of UAS within the European Union. This regulation is a key component of the EU's broader drone legislation and forms a crucial step towards harmonizing drone regulations across the EU. It sets specific requirements for the design and manufacturing of drones, with further practical details provided in the complementary regulation 2019/947.

Regulation 2019/945 outlines general rules regarding RID. Each UAS must provide a direct periodic broadcast using an open and documented transmission protocol. The broadcast must include information such as:
- The operator registration number.
- The unique serial number of the UAS.
- Geographical coordinates, including altitude and heading.
- The take-off point (if the location of the remote pilot is unavailable).

Further details can be found in Part 6 of the Annex of the regulation. Note that these regulations don't specify any details on _how_ to implement remote identification.
### European standard
The European Standard EN 4709-002 serves as a guiding document for regulation 2019/945, offering rules and guidelines for implementing DRI. Developed by ASD-STAN and CEN, this standard outlines general requirements including message formats, transport protocols, and output power specifications.

While the standard does not specify a particular transport method for RID messages, most frameworks that adhere to this standard utilize both Wi-Fi and Bluetooth technologies for communication.

### Implementation and enforcement
The European Union Aviation Safety Agency (EASA) is responsible for certification, regulation, and standardization of civil aviation safety. EASA oversees the implementation and enforcement of regulation 2019/945 and provides guidance materials and technical standards to support safe integration of drones in the EU airspace.

In summary, these regulations and standards create a framework for the integration of drones into the EU airspace, promoting safety, efficiency, and technological advancement.
## Implementation
### OpenDroneID
OpenDroneID (ODID) is a standardized, open-source solution for implementing RID systems in UAS. Compliant with the ASD-STAN EN 4709-002 Direct Remote ID specifications, ODID adheres to the European regulation 2019/945, ensuring a reliable RID system for drones.

The framework provides a method for transmitting and receiving RID messages. It details how data is encoded and packed, enabling compatible receivers to decode and interpret the messages. Being developed as open source means it is designed to work across various platforms and devices, allowing for easy integration into existing infrastructure.

OpenDroneID also provides documentation and support to assist developers and manufacturers in deploying the system. Furthermore, ODID is fully compatible with MAVLink (see next section) which enhances easy deploying on various UAS applications.

One of the primary advantages of ODID is its support for microcontrollers with Bluetooth and/or Wi-Fi capabilities, such as the ESP32 series, which enables the RID transmitter to be low cost.
### MAVLink
MAVLink (Micro Air Vehicle Link) is a lightweight communication protocol designed for the exchange of messages between unmanned vehicles, such as drones, and ground control stations (GCS) or other onboard systems. MAVLink operates over various transport layers, including UDP, TCP, and serial connections. The protocol is widely supported by popular autopilot systems such as ArduPilot and PX4.
#### MAVLink messages
The protocol defines a set of standard messages and provides a framework for creating custom messages (a.k.a. dialects) enabling developers to create applications for flight control, telemetry, mission planning, and more. By using a binary message format it reduces bandwidth usage and improves communication speed and reliability. This makes MAVLink suitable for aircraft systems where low latency and efficiency are important.

#### MAVLink setup
A typical MAVLink setup includes a drone equipped with a flight controller, responsible for flight operations, and a companion computer that handles mission planning, computer vision, and other complex tasks. The ground control station utilizes ground control software, such as QGroundControl or Mission Planner, to provide an interface for users to monitor and control the drone.

The communication link between the GCS and the drone usually consists of 2 telemetry radios operating on the 433 MHz frequency. Onboard MAVLink communication occurs via local bus systems such as UART or DroneCAN.
#### Remote ID support
OpenDroneID messages are part of the standard MAVLink message set, with each identification feature represented by its own message. These include messages for Operator ID, drone location, operator location, and other essential features.

### Implementing ODID with MAVLink
The remote identification features that need to be sent by the RID transmitter are stored across different components of the drone system, making MAVLink a valuable tool for integrating this data.
- **The flight controller** holds data specific to the drone itself, such as location, speed, heading, and the drone's serial number.
- **The ground control station** holds information like the operator ID and operator's location or take-off point.

By routing all this data through the MAVLink network to the RID transmitter equipped with OpenDroneID, the information can be encoded and broadcasted over Bluetooth and Wi-Fi to other OpenDroneID receivers, such as smartphones with compatible OpenDroneID apps.
## Challenges and Limitations
### Security and Privacy
Direct Remote Identification introduces several security and privacy challenges that must be addressed to ensure safe and compliant drone operations. One major privacy concern is the public accessibility of the operator’s location, potentially compromising the privacy of the drone pilot. DRI mitigates this by limiting access to individuals within the immediate broadcast range who may be directly impacted by the drone’s operation. However, there remains a risk that receivers could publish this location data online on platforms like FlightAware or Flightradar24, increasing potential privacy risks.

Another concern is that Bluetooth and Wi-Fi signals, used for DRI broadcasts, are relatively easy to spoof, potentially allowing malicious actors to manipulate the DRI broadcasts. To address the risk of unauthorized use of an operator’s ID, operator IDs include a unique checksum derived from a hidden three-character sequence known only to the registered individual. This checksum must be accurately calculated upon entry, ensuring that the ID is valid.

### Interference and Range Issues
Since Wi-Fi and Bluetooth signals are affected by obstacles, Remote ID broadcasts may experience interference when the signal encounters buildings, trees, or other structures. Drones flying in urban areas, for example, may have limited reach of their remote ID signal due to the effects of multipath interference, where signals reflect off various surfaces.

Direct Remote ID broadcasts often use unlicensed frequency bands (e.g. 2.4Ghz and 5GHz for Wi-Fi and Bluetooth), which are already heavily used by other devices. This can lead to signal congestion, increasing the likelihood of interference and reducing the reliability of RID broadcasts. If multiple drones are broadcasting Remote ID data, the overlapping signals may interfere with each other, making it difficult for receivers to correctly identify and separate individual drone transmissions.

Network Remote ID (NRI) addresses some of the range and interference limitations of DRI by enabling data upload to the internet, enabling Beyond Visual Line of Sight (BVLOS) identification. However, this method introduces additional privacy concerns for pilots and requires a more complicated setup.
## Future and Research Opportunities
The regulation of UAS in the European Union is still in the early stages, presenting numerous opportunities for improvement, especially regarding the use of remote identification systems.

One promising development is the upcoming Drone Remote Identification Protocol (DRIP) by the Internet Engineering Task Force (IETF). While OpenDroneID adheres to the EN 4709-002 standard, DRIP builds upon this foundation and adds enhanced safety and security features. These improvements create a more robust framework for drone identification. However, as DRIP is still in the early stages of development, the proven and established OpenDroneID remains the more widely used framework.

In addition to transmitting identification information, RID messages can also be received by drones to enable functionalities such as situation awareness. By processing the location and movement data of nearby drones, a UAS can adjust its flight path to prevent potential collisions. This capability not only helps to prevent accidents but also helps to create a collaborative airspace where drones can operate alongside one another.

## Conclusion
The implementation of remote identification for drones represents a significant step towards a safe integration of UAS into airspace. By requiring real-time identification of drones, these regulations promote pilot accountability, enhance public safety, and improve community acceptance of drone operations.

Technologies such as OpenDroneID and MAVLink provide robust frameworks for implementing RID systems into existing setups. Additionally, ongoing research like the Drone Remote Identification Protocol (DRIP) promises to enhance the security and reliability of remote identification mechanisms. However, challenges still remain, including concerns relating to security, privacy violations, and issues of signal interference and range.

As UAS technology continues to evolve, the integration of situational awareness helps to create a future where drones can coexist within shared airspace. Ultimately, the successful implementation of remote identification represents not just a regulatory requirement, but a crucial advancement toward a safe drone ecosystem.
