# Executive Summary

This technical document provides a step-by-step analysis of a network packet capture (PCAP) investigation completed as part of the *Cisco Networking Academy* curriculum. The primary objective of this exercise was to identify, isolate, and characterize a compromised host endpoint on a local subnet using *Wireshark* display filters, OSI protocol layer inspection, and header payload decoding.

Wireshark functions much like an *Airport Luggage Scanner* for network communications: rather than simply checking the routing label on the outside of a package (Layer 3/4 headers), it enables security analysts to inspect the payload, protocol behaviors, and precise byte-level parameters hidden within passing frames.

---

## Technical Analysis & Walkthrough

### Phase 1: Noise Reduction & Traffic Filtering

High-traffic networks produce thousands of frames per second, making manual frame-by-frame analysis inefficient during incident response.

* *Protocol Filtering:* Applied specific display filters (dhcp and http) to eliminate background broadcast noise and isolate dynamic IP assignments and web application requests.
* *DHCP Identification:* Isolating DHCP traffic allows an analyst to observe network lease requests, identifying when new hosts join the subnet or request IP updates.

text
Display Filter: dhcp || http


![DHCP WIRESHARK](Screenshot-032228.png)



---

### Phase 2: Target Isolation & Endpoint Identification

After narrowing down the PCAP data, individual frames were interrogated to identify the target IP address and endpoint hardware details.

#### 1. Target IP Isolation

* *Frame Examined:* Frame 3020.
* *Protocols Inspected:* Ethernet II (Layer 2) and Internet Protocol Version 4 (Layer 3).
* *Findings:* Inspection of the IPv4 header fields within Frame 3020 confirmed the source address of the affected/infected machine:
* *Affected Machine IP:* 172.16.165.165



#### 2. Hostname Extraction

* *Payload Layer:* DHCP Option 12 / HTTP User-Agent / SMB Session Setup.
* *Findings:* Deep packet inspection of protocol payloads linked to 172.16.165.165 revealed the active Windows hostname:
* *Infected Endpoint Hostname:* K34ENGW3N-PC



---

### Phase 3: Byte-Level Protocol & Header Decoding

Accurate threat detection requires verifying protocol header fields at the bit and byte level.

#### 1. IP Version Header Verification

By inspecting the first nibble (4 bits) of the IP header in hex/binary representation, the IP protocol version is determined:

* 0100 (Binary) = *IPv4* (4 in decimal)
* 0110 (Binary) = *IPv6* (6 in decimal)

#### 2. Media Access Control (MAC) & OUI Analysis

* *MAC Address Anatomy:* A MAC address consists of 12 hexadecimal digits (48 bits).
* *OUI Decoding:* The first 6 hexadecimal digits (24 bits) represent the *Organizationally Unique Identifier (OUI)* assigned to the Hardware Vendor/Manufacturer.
* *Application:* Extracting the OUI allows security analysts to verify whether an endpoint is an expected enterprise device (e.g., Dell, Cisco, Intel) or an unauthorized rogue device.



![Affected Machine and Host](Screenshot-2026-09-22-032811.png)

---

## Investigation Summary Table

| Parameter | Value / Finding | Analysis Method |
| --- | --- | --- |
| *Primary Investigation Tool* | Wireshark | PCAP File Analysis |
| *Key Inspection Frame* | Frame 3020 | Layer 2/3 Header Inspection |
| *Affected Endpoint IP* | 172.16.165.165 | IPv4 Source Field Mapping |
| *Compromised Hostname* | K34ENGW3N-PC | DHCP/Application Payload Analysis |
| *Protocol Header Check* | 0100 (IPv4) / 0110 (IPv6) | Binary Header Nibble Verification |
| *Vendor Tracking Method* | MAC OUI (First 6 Hex Digits) | Hardware Manufacturer Lookup |

---

## Practical SOC Takeaways

1. *Structured Filtering Saves Time:* Relying on targeted protocols (dhcp, http, dns) prevents analysis fatigue and reduces the Mean T…
