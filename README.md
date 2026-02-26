# Enterprise Network with High Availability and Integrated Linux Services

## 1. Project Overview

This project simulates a resilient, medium-sized enterprise network in GNS3, focusing on High Availability (HA) and the integration of core infrastructure services. The design ensures business continuity by eliminating single points of failure at both the routing (Layer 3) and switching (Layer 2) layers.
Key technologies implemented include VRRP for gateway redundancy, OSPF for dynamic routing, LACP for link aggregation, and a centralized Ubuntu Server providing essential services like DHCP and NTP.

![Topology Diagram](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/Topology.png)

## 2. IP Addressing and VLAN Plan
The network is logically segmented using VLANs to isolate traffic and enhance security. A virtual IP (VIP) provided by VRRP serves as the redundant gateway for all devices.

#### IP Addressing and Subnet Overview
| Device / Interface                  | IP Address                       | Subnet Mask     | Default Gateway       |
| ----------------------------------- | -------------------------------- | --------------- | --------------------- |
| ISP Router Gi0/5                    | 192.0.2.1                        | 255.255.255.252 | N/A                   |
| Router R1 Gi0/5                     | 192.0.2.2                        | 255.255.255.252 | N/A                   |
| ISP Router Gi0/4                    | 192.0.2.5                        | 255.255.255.252 | N/A                   |
| Router R2 Gi0/4                     | 192.0.2.6                        | 255.255.255.252 | N/A                   |
| ISP Router Gi0/0                    | 100.64.10.1                      | 255.255.255.252 | N/A                   |
| Internet Host e0                    | 100.64.10.2                      | 255.255.255.252 | 100.64.10.1           |
| Router R1 Gi0/0 and Router R2 Gi0/0 | R1: 10.90.200.1, R2: 10.90.200.2 | 255.255.255.252 | N/A                   |
| Xubuntu-Server                      | 10.90.10.10                      | 255.255.255.192 | 10.90.10.1 (VRRP VIP) |
| Xubuntu-Admin                       | 10.90.20.10                      | 255.255.255.192 | 10.90.20.1 (VRRP VIP) |
| Office PC                           | 10.90.30.100                     | 255.255.254.0   | 10.90.30.1 (VRRP VIP) |
| Warehouse PC                        | 10.90.40.100                     | 255.255.254.0   | 10.90.40.1 (VRRP VIP) |
| Switches SW1-SW4 (VLAN 99 SVI)      | 10.90.99.1                       | 255.255.255.240 | 10.90.99.2 (VRRP VIP) |

#### VLAN Planning
| VLAN ID | VLAN Name  | Default Gateway (VRRP VIP) | R1 Subinterface IP | R2 Subinterface IP | Assigned Devices   | Switch Assoc       |
| ------- | ---------- | -------------------------- | ------------------ | ------------------ | ------------------ | ------------------ |
| 10      | Server     | 10.90.10.1                 | 10.90.10.2         | 10.90.10.3         | Xubuntu-Server     | SW3                |
| 20      | Admin      | 10.90.20.1                 | 10.90.20.2         | 10.90.20.3         | Xubuntu-Admin      | SW3                |
| 30      | Office     | 10.90.30.1                 | 10.90.30.2         | 10.90.30.3         | Office PCs         | SW4                |
| 40      | Warehouse  | 10.90.40.1                 | 10.90.40.2         | 10.90.40.3         | Warehouse PCs      | SW4                |
| 99      | Management | 10.90.99.1                 | 10.90.99.2         | 10.90.99.3         | SW1, SW2, SW3, SW4 | SW1, SW2, SW3, SW4 |



## 3. High Availability (HA) Architecture

The network was designed from the ground up for resilience and continuous operation.
Layer 3 Redundancy: VRRP & OSPF
Redundant Gateway: VRRP was configured to provide a virtual, fault-tolerant default gateway for all VLANs. R1 serves as the Master router, with R2 acting as a Backup, ready to take over instantly if the primary fails.
Dynamic Routing: OSPF was deployed as the Interior Gateway Protocol (IGP). This ensures intelligent, dynamic path selection and rapid convergence, automatically rerouting traffic if a link or router goes down.
Layer 2 Stability: LACP & RSTP
Link Aggregation: LACP EtherChannel was used to bundle multiple physical links between switches. This increases total bandwidth and provides link redundancy.
Loop Prevention: RSTP (Rapid Spanning Tree Protocol) was enabled on all switches to prevent catastrophic Layer 2 loops while ensuring sub-second convergence times.

## 4. Centralized Services on Ubuntu Server
An Ubuntu Server 20.04 was integrated into the network (VLAN 10) to centralize and manage core services.
Centralized DHCP: An ISC-DHCP-Server was installed to manage IP address allocation for all client VLANs. Router interfaces were configured as DHCP Relay Agents (ip helper-address) to forward client requests to the server.
Time Synchronization (NTP): The server was configured as a master NTP server using Chrony. All network devices synchronize their clocks with this server, ensuring accurate timestamps in logs.
Secure Management: SSH v2 was enforced for all device management, using 2048-bit RSA keys to encrypt administrative sessions.

## 5. Verification and Documentation
This section provides screenshots that validate the configuration and functionality of the network.

### 5.1 Infrastructure Health (Routing & Switching)
- OSPF Neighbor Adjacency: Confirmation of full neighbor states between core routers.

![](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/OSPF.png)

- Spanning Tree (RSTP): Verification of loop-free topology and rapid convergence states.

![](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/RSTP.png)

- Link Aggregation (LACP): Bundling physical interfaces for bandwidth and redundancy.

![](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/LACP.png)

- NAT/PAT Operation: Validation of internal traffic translation to the public internet address space.

![](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/NAT%26PAT.png)

- Secure Management (SSH): Demonstrating secure remote access from Router R1 to Switch SW4 via the Management VLAN.

![](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/join%20ssh%20from%20R1%20to%20SW4-2.png)


### 5.2 Gateway Redundancy (VRRP)
This sub-section validates the failover mechanism between the Master and Backup routers.

  - Initial VRRP State: R1 operating as the Master for all VLANs.

![](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/VRRP.png)

  - Failover Scenario: Verification of R2 transitioning to Master after an intentional shutdown of R1.

![](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/drop%20R1%20-%20R2%20master%20VRRP.png)

  - Live Redundancy Test (Video): A screen recording demonstrating zero-downtime (or minimal packet loss) during a gateway failure.

![](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/vlc-record-2026-02-26-03h13m36s-VRRP%20test.mp4)

### 5.3 Core Services (DHCP & NTP)
Centralized services hosted on the Ubuntu Server (VLAN 10) and managed across the infrastructure.

- DHCP Server Operations:
  - Server status and binding logs:
    
![](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/dhcp%20servers.png)

  - DHCP Pool configuration summary:
   
![](https://github.com/abd-alhusseini/Project-2/blob/main/Screenshots/server%202.png)

  - Client lease verification for VLAN 40 (Warehouse):
    
![](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/dhcp%20vl%2040%20.png)

  - Real-time DORA process monitoring:
    
![](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/DHCP.png)

- NTP Synchronization:
  - NTP access control configuration (Allowing 10.90.0.0/16):
    
![](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/NTP%20services%20Allow%2010.90.0.0.png)

  - Time synchronization tracking between server and network devices:
    
![](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/NTP.png)

### 5.4 Protocol Deep-Dive (Wireshark Analysis)
Detailed packet-level inspection to confirm protocol compliance and health.

  - OSPF Packet Analysis: Monitoring Hello packets and LSA exchanges.
    
![](https://github.com/abd-alhusseini/Project-2/raw/main/Screenshots/wir-ospf.png)

  - VRRP Packet Analysis: Verifying periodic advertisements and virtual MAC health.

![](https://github.com/abd-alhusseini/Project-2/blob/main/Screenshots/Wireshark-VRRP.png)


## 6. Skills Demonstrated
Advanced Routing & Redundancy: OSPF, VRRP, LACP EtherChannel, RSTP.
Network Services Integration: DHCP Relay, NTP, and NAT-on-a-Stick.
Linux System Administration: Ubuntu Server, ISC-DHCP-Server, Chrony.
Network Security & Hardening: Secure management with SSH, VLAN segmentation.
Protocol Analysis: Packet analysis with Wireshark and verification using Cisco IOS commands.

## 7. Technologies Used
Simulation Environment: GNS3 (v2.2.56.1)

Network Analysis: Wireshark (v 4.4.4) 

Core Routers: Cisco C7200 Series 

Access Switches: Cisco IOSv-L2 

Server Infrastructure: Ubuntu Server 20.04.4 LTS (QEMU VM)

Security & Testing: Kali Linux (Desktop Environment)

End-User Workstations: VPCS (Lightweight PC Simulator)
