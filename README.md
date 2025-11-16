# Network Segmentation & VLAN Configuration  
Router-on-a-Stick • DHCP • VLANs • Cisco Packet Tracer

[![Cisco IOS](https://img.shields.io/badge/Platform-Cisco_IOS-1ba0d7?logo=cisco)](https://www.cisco.com/)
[![Packet Tracer](https://img.shields.io/badge/Tool-Packet_Tracer-0a7cff?logo=cisco)](https://www.netacad.com/courses/packet-tracer)
[![Routing & Switching](https://img.shields.io/badge/Focus-Routing_&_Switching-green)](#)
[![DHCP](https://img.shields.io/badge/Service-DHCP_Automation-yellow)](#)
[![VLANs](https://img.shields.io/badge/Feature-VLAN10-orange)](#)

---

## Project Overview

This project demonstrates how to configure a segmented Layer 2 network with DHCP, VLANs, trunking, and router-on-a-stick using Cisco Packet Tracer. A single router provides Layer 3 services for the Marketing VLAN, while a Layer 2 switch handles VLAN membership and 802.1Q trunking. The PC in VLAN 10 receives its network configuration dynamically via a DHCP pool hosted on the router.

The goal is not only to show the final working configuration, but also to walk through the real troubleshooting steps and decisions made along the way; mirroring how network engineers diagnose and fix issues in live environments. You can download and explore the completed Packet Tracer file below:

**[Network Segmentation & VLAN Configuration](Network%20Segmentation%20%26%20VLAN%20Configuration.pkt)**

---

## 1. Logical Topology & Planning

The environment contains:

- **Router0** → Provides routing and DHCP  
- **Switch0** → Handles VLANs and trunking  
- **PC0** → Client in VLAN 10, set to obtain IP via DHCP

<img width="1920" height="1032" alt="S1" src="https://github.com/user-attachments/assets/3d50010f-e3a6-474e-99cf-e3b490624148" />

> **Cabling Used:**  
> - Router ↔ Switch → Straight-through (different device types)  
> - Switch ↔ PC → Straight-through  

**VLAN / IP Design**

| VLAN | Purpose    | Subnet           | Gateway         |
|------|-------------|------------------|-----------------|
| 10   | Marketing   | 192.168.10.0/24  | 192.168.10.1    |
| 20   | HR          | (Reserved)       | (Planned)       |

---

## 2. Router Initial Configuration (S2)

Upon opening the router, Packet Tracer displays the **System Configuration Dialog**.  
We intentionally selected **“no”** so that we could build the configuration manually. This allows full granular control of every interface, service, and protocol — matching what real network engineers do.

<img width="702" height="712" alt="S2" src="https://github.com/user-attachments/assets/a272acdd-4ba3-4baa-9d69-837850ebb2d6" />

We entered privileged **EXEC** mode using the command:

- **`enable`**  
  - Moves from *user EXEC* (`Router>`) to *privileged EXEC* (`Router#`)  
  - Required for viewing system-level information and entering configuration mode  
  - No parameters

Next, we entered global configuration mode:

- **`configure terminal`**  
  - Abbreviated as `conf t`  
  - Allows applying all system-wide and interface-specific changes  
  - No parameters

We then configured the physical interface:

- **`interface gigabitEthernet0/0/0`**  
  - Opens the configuration context for that specific interface  
  - Interface IDs follow: `type slot/port/subport`

Inside the interface we used:

- **`ip address <address> <mask>`**  
  Assigns an IP address to the interface. Example:  
  `ip address 192.168.1.1 255.255.255.0`

- **`no shutdown`**  
  - Brings the interface up  
  - All Cisco interfaces default to *administratively down*

Although this activated the interface, the IP (`192.168.1.1`) did **not** match our intended VLAN 10 design — which ultimately contributed to the ping failure observed later.

---

## 3. DHCP Configuration (S3)

With the router active, we configured DHCP exclusions and created a dedicated DHCP pool.

<img width="702" height="712" alt="S3" src="https://github.com/user-attachments/assets/bb0eca13-c95c-4a8b-92b4-7f913e978cfa" />

### Commands Introduced Here

- **`ip dhcp excluded-address <start> <end>`**  
  Prevents DHCP from assigning addresses in a reserved range.  
  We used:  
  - `ip dhcp excluded-address 192.168.10.1`  
  - `ip dhcp excluded-address 192.168.10.10`  

- **`ip dhcp pool <name>`**  
  Creates a DHCP pool and enters DHCP configuration mode.  
  We named ours `VLAN10_Pool`.

Inside the DHCP pool:

- **`network <ip> <mask>`**  
  Defines the subnet the pool will use.

- **`default-router <ip>`**  
  Sets the gateway handed out to clients — must match router-on-a-stick subinterface.

- **`dns-server <ip>`**  
  Provides a DNS server address to DHCP clients.

After configuration we verified bindings using:

- **`show ip dhcp binding`**  
  - EXEC-level command  
  - Displays active DHCP leases  
  - Failed at first because it was incorrectly run in config mode

---

## 4. VLAN Creation & Port Assignment (S4)

Next, we configured the Layer 2 switch.

<img width="702" height="712" alt="S4" src="https://github.com/user-attachments/assets/c35f99c2-5b3a-4478-bee2-05d7dc77b86d" />

### Commands Introduced Here

- **`vlan <id>`**  
  Creates (or opens) a VLAN configuration context.  
  We created:  
  - `vlan 10` → named *Marketing*  
  - `vlan 20` → named *HR*

- **`interface fastEthernet0/2`**  
  Enters the interface to assign it to a VLAN.

Inside the interface:

- **`switchport mode access`**  
  Forces the port into access mode.

- **`switchport access vlan <id>`**  
  Assigns the VLAN to the interface (untagged).

At this point the VLANs were created and the PC was assigned to VLAN 10, but trunking and subinterfaces were not yet configured — so routing still could not occur.

---

## 5. L2 Verification (S5.1, S5.2, S5.3)

Before testing connectivity, we verified Layer 2 status.

### **Switch Interface Summary (S5.1)**  
<img width="702" height="712" alt="S5 1_Results" src="https://github.com/user-attachments/assets/366fbc36-eaa0-4ec5-993c-5e0ff537c04b" />

`show ip interface brief` confirmed the physical links were up.

### **VLAN Table (S5.2)**  
<img width="702" height="712" alt="S5 2_Results" src="https://github.com/user-attachments/assets/0ee564ff-409f-476a-8507-14c423e9d33c" />

`show vlan brief` showed VLAN 10 correctly mapped to `Fa0/2`.

### **Trunk Capability (S5.3)**  
<img width="702" height="712" alt="S5 3_Results" src="https://github.com/user-attachments/assets/595de571-e9a9-4c02-ae23-e35a7f591a10" />

`show interfaces trunk` confirmed that `Fa0/1` was able to operate as a trunk, but still needed configuration.

---

## 6. First Connectivity Test – Ping Failure (S6)

We attempted to ping the router.

<img width="702" height="712" alt="S6" src="https://github.com/user-attachments/assets/0187f75c-d972-45a4-bd2b-62ece95726a7" />

The PC failed to reach `192.168.1.1`, which made sense once we realized:

- The DHCP pool was for **192.168.10.0/24**  
- The router’s interface was on **192.168.1.0/24**  
- VLAN 10’s gateway **should be 192.168.10.1**

This mismatch pointed directly to the missing router subinterface.

---

## 7. Creating the Router-on-a-Stick Subinterface (S7)

We aligned routing with the VLAN design by creating a subinterface.

<img width="702" height="712" alt="S7" src="https://github.com/user-attachments/assets/33e33ee1-949d-4b00-9366-cffe12432720" />

### Commands Introduced Here

- **`interface gigabitEthernet0/0/0.<vlan>`**  
  Creates a logical subinterface for a VLAN.  
  Example: `interface g0/0/0.10`

- **`encapsulation dot1Q <vlan-id>`**  
  Enables 802.1Q tagging on the subinterface.  
  Example: `encapsulation dot1Q 10`

- **`ip address <ip> <mask>`**  
  Assigns the gateway for that VLAN.

This corrected the addressing mismatch and completed the router side of the router-on-a-stick setup.

---

## 8. Updating the Switch Trunk Configuration (S8)

Next, we configured the switch uplink so traffic could reach the subinterface.

<img width="702" height="712" alt="S8" src="https://github.com/user-attachments/assets/24e459a1-c2cd-4fd8-b6a2-b97011a127a8" />

### Commands Introduced Here

- **`switchport mode trunk`**  
  Converts the interface to trunking mode.

- **`switchport trunk allowed vlan <list>`**  
  Specifies which VLANs are allowed on the trunk.

Once the trunk matched the router subinterface, inter-VLAN communication became possible.

---

## 9. Final PC Configuration Confirmation

This image shows the PC set explicitly to **DHCP**, which is required for it to receive the correct addressing information.

<img width="702" height="712" alt="S10" src="https://github.com/user-attachments/assets/363e828e-3060-42db-bb27-98185406943a" />

> The PC was previously left in static mode during testing; this alone would have prevented DHCP from assigning an address. Changing it to DHCP ensured proper VLAN participation.

---

## 10. Successful Ping & DHCP Assignment

With the gateway corrected and trunking finalized, the PC obtained a proper DHCP lease.

<img width="702" height="712" alt="S9" src="https://github.com/user-attachments/assets/f81f64ea-c118-4300-9e59-35992b2d6003" />

- Assigned IP: `192.168.10.2`  
- Default Gateway: `192.168.10.1`  
- DNS: `8.8.8.8`  

Ping to `192.168.10.1` was successful, confirming full network functionality.

---

## Conclusion

This project demonstrated a complete workflow for building and validating a segmented network with VLANs, DHCP, and router-on-a-stick routing. Beyond applying commands, it walked through real troubleshooting steps: identifying addressing mismatches, correcting subinterfaces, enabling trunking, and validating end-to-end connectivity. The final working environment behaves exactly like a small enterprise network, with scalable VLAN architecture and centralized Layer 3 routing.

