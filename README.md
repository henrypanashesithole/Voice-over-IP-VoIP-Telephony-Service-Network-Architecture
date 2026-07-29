# Voice over IP (VoIP) & Telephony Service Network Architecture

A complete multi-site enterprise Telephony & IP Networking design engineered and simulated in Cisco Packet Tracer. This project demonstrates centralized and local DHCP services, Inter-VLAN routing (Router-on-a-Stick), multi-router OSPF dynamic routing, Cisco Unified Communications Manager Express (CME) telephony integration, and VoIP Dial-Peering across departmental sites.

---

## Why I Did This Project (Project Purpose)

In modern enterprise communications, legacy PSTN systems are costly, rigid, and inefficient. Migrating to an IP-based Telephony infrastructure allows organizations to run voice and data over a unified network, reducing telecom costs while providing scalable, high-quality communications across departments.

**The main objectives of this project were to:**
* **Implement IP Telephony Across Sites:** Enable seamless voice calls between distinct departments (ICT, Finance, HR, and Sales) using Cisco IP Phones (CP-7960).
* **Separate Voice & Data Traffic:** Segregate voice streams from standard computer traffic using dedicated Voice VLANs to guarantee Quality of Service (QoS) and network security.
* **Configure VoIP Dial-Peering:** Establish Inter-Site VoIP call routing across WAN routers using VoIP dial-peers and target IP address mappings.
* **Automate Endpoint Provisioning:** Utilize DHCP Option 150 to automatically push TFTP boot information and router IP source addresses to IP phones upon startup.

---

## Key Technologies Implemented & "The Why"

| Technology | Implementation Scope | Why We Implemented It (Technical Rationale) |
| :--- | :--- | :--- |
| **Cisco CME (`telephony-service`)** | All Edge/Core Routers | Provides local call-processing features, `ephone-dn` directory number assignments, and ephone MAC-address registration directly on Cisco ISR 2811 routers. |
| **VoIP Dial Peers** | Gateway Routers | Maps destination phone number patterns (e.g., `1..`, `2..`, `3..`, `4..`) to next-hop router IP addresses over the OSPF WAN backbone for cross-site calling. |
| **DHCP Option 150** | Voice DHCP Pools on Routers | Instructs Cisco IP Phones where to find the CME TFTP server/IP source address to register and download configurations automatically. |
| **Voice & Data VLAN Isolation** | Access Switches & Trunks | Configured `switchport voice vlan 100` alongside data VLANs on switch ports to ensure auxiliary voice tagging and prevent data congestion from degrading call quality. |
| **Router-on-a-Stick (RoaS)** | Subinterfaces on FA0/0 & FA0/1 | Provides cost-effective Layer 3 routing between local department Data VLANs, Voice VLANs, and the Central Server Room. |
| **OSPF Area 0 Dynamic Routing** | All Site Routers | Propagates data and voice subnets dynamically across serial point-to-point links (`10.10.10.0/30`), ensuring reachable WAN paths for VoIP packets. |
| **DHCP Relay (`ip helper-address`)** | Subinterfaces pointing to `192.168.100.130` | Relays client IP lease requests from local Data VLANs to the centralized DHCP server in the Server Farm. |

---

## Key Takeaways & What I Learned

1. **Importance of Voice VLAN Tagging:** Configuring `switchport voice vlan 100` alongside standard access VLANs allows a single Ethernet port to carry untagged PC data and tagged VoIP voice traffic through an IP Phone's internal switch.
2. **Dial Plan Design & Matching:** Constructing wildcards in `destination-pattern` commands (e.g., `1..` for Finance 100s, `2..` for ICT 200s, `3..` for Sales 300s, `4..` for HR 400s) streamlines call routing across distributed routers.
3. **Automated Phone Registration:** Understanding how Cisco IP Phones sequence boot stages—requesting an IP via DHCP, reading Option 150, connecting to port 2000 (`Skinny Client Control Protocol - SCCP`), and fetching their DN line key configuration.
4. **Physical Rack Organization:** Building out a physical rack with patch panels, cable conduits, PDUs, and dedicated 2811 ISR routers bridges the gap between theoretical CLI commands and hardware setup.

---

## Network Architecture & Topology Breakdown

The infrastructure consists of 4 main departmental sites and 1 centralized Server Room connected via `/30` serial point-to-point links:

```text
       [ ICT Router ] (10.10.10.1 / 10.10.10.5)
             |
             +---- [ FINANCE Router ] (10.10.10.2 / 10.10.10.9)
             |           |
             +---- [ HR Router ]      (10.10.10.10 / 10.10.10.14)
             |           |
             +---- [ SALES Router ]   (10.10.10.6 / 10.10.10.13)

1. ICT Department (Core Site):

- Data VLAN 40 (192.168.100.96/27) | Gateway: 192.168.100.97

- Voice VLAN 100 (172.16.100.96/27) | Gateway/TFTP: 172.16.100.97

- Extension Range: 201 – 210

2. Finance Department:

- Data VLAN 10 (192.168.100.0/27) | Gateway: 192.168.100.1

- Voice VLAN 100 (172.16.100.0/27) | Gateway/TFTP: 172.16.100.1

- Extension Range: 101 – 110

HR Department:

Data VLAN 20 (192.168.100.32/27) | Gateway: 192.168.100.33

Voice VLAN 100 (172.16.100.32/27) | Gateway/TFTP: 172.16.100.33

Extension Range: 401 – 410

3. Sales Department:

- Data VLAN 30 (192.168.100.64/27) | Gateway: 192.168.100.65

- Voice VLAN 100 (172.16.100.64/27) | Gateway/TFTP: 172.16.100.65

- Extension Range: 301 – 310

4. Server Room Site:

- Data VLAN 50 (192.168.100.128/29)

- Central DHCP Server (192.168.100.130), DNS (192.168.100.131), Email, and Web Servers.

IP Addressing & Dial Plan Chart

| Site / Department | Data Subnet | Voice Subnet | Voice Pool Router IP | Dial Pattern | Ext. Range |
| :--- | :--- | :--- | :--- | :---: | :---: |
| **Finance** | `192.168.100.0/27` | `172.16.100.0/27` | `172.16.100.1` | `1..` | 101 - 110 |
| **ICT** | `192.168.100.96/27` | `172.16.100.96/27` | `172.16.100.97` | `2..` | 201 - 210 |
| **Sales** | `192.168.100.64/27` | `172.16.100.64/27` | `172.16.100.65` | `3..` | 301 - 310 |
| **HR** | `192.168.100.32/27` | `172.16.100.32/27` | `172.16.100.33` | `4..` | 401 - 410 |
| **Server Room** | `192.168.100.128/29` | N/A | N/A | N/A | N/A |

Key Configuration Highlights
1. Telephony Service & Option 150 Configuration (ICT Router)

ip dhcp excluded-address 172.16.100.97

ip dhcp pool ITVOICEPOOL
 network 172.16.100.96 255.255.255.224
 default-router 172.16.100.97
 option 150 ip 172.16.100.97
 dns-server 172.16.100.97

telephony-service
 max-ephones 20
 max-dn 20
 ip source-address 172.16.100.97 port 2000
 auto assign 1 to 20

ephone-dn 1
 number 201

2. VoIP Dial-Peering Across WAN Routers

! Dial Peer toward Finance (Extensions 1xx)
dial-peer voice 2 voip
 destination-pattern 1..
 session target ipv4:10.10.10.2

! Dial Peer toward Sales (Extensions 3xx)
dial-peer voice 6 voip
 destination-pattern 3..
 session target ipv4:10.10.10.6

! Dial Peer toward HR (Extensions 4xx)
dial-peer voice 5 voip
 destination-pattern 4..
 session target ipv4:10.10.10.14

3. Switchport Voice VLAN Access Layer Configuration

interface FastEthernet0/2
 switchport access vlan 40
 switchport mode access
 switchport voice vlan 100

Verification & Test Results
The implementation was validated using Cisco Packet Tracer GUI and CLI tools:

1. Central Data DHCP Assignment: Hosts on Data VLANs successfully requested IP leases from the Central Server (192.168.100.130) via ip helper-address. Example: PC0 assigned 192.168.100.105/27, Gateway 192.168.100.97, DNS 192.168.100.131.

2. IP Phone Registration: Cisco CP-7960 phones automatically obtained IPs from local voice pools and registered line extensions via SCCP (Port 2000).

3. Cross-Site VoIP Calling Test:
- Initiated call from Extension 207 (ICT Site) to Extension 203 (Remote Site).
- Result: "Ring Out" displayed on calling device TO  "Connected" established upon answer with bi-directional audio path verified over OSPF WAN.

How to Run & Test
- Launch Cisco Packet Tracer.

- Open the .pkt file for Project 8.

- Allow OSPF and STP time to converge (~30 seconds).

- Click on IP Phone 37 (Extension 207) and navigate to the GUI tab.

- Dial extension 203 (or any configured ephone extension) and click the handset to verify Ring Out / Connected status.
