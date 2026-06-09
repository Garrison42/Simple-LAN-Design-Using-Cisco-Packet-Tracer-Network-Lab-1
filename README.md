# Simple LAN Design Using Cisco Packet Tracer | Network Lab #1
## Watch me build this lab on YouTube => 

## Network Lab #1 – VLANs, DHCP & Wireless Networking

## Overview

This project demonstrates the design and implementation of a branch office network for **Simon Enterprises**. The branch network was designed to operate independently from the headquarters infrastructure while providing secure and reliable connectivity for three departments:
* IT
* Accounts
* Sales

This lab demonstrates:
* Network Design & Planning
* Cisco Switch Configuration
* Cisco Router Configuration
* VLAN Implementation
* 802.1Q Trunking
* Router-on-a-Stick
* DHCP Deployment
* Wireless Networking
* IPv4 Addressing
* Subnetting
* Connectivity Testing

# Business Scenario

Simon Enterprises is a rapidly expanding company based in Western Canada with over 1.5 million customers worldwide. The company specializes in the distribution and retail of hardware and construction supplies, managed centrally from its headquarters. The company plans to establish a new branch near the town of Maplewood. As part of this expansion, the company is requiring a network engineer to design and implement the branch network. The branch network is intended to operate independently from the HQ infrastructure.

# Network Requirements

The company specified the following requirements:

1.	One router and one switch to be used (all CISCO products).
2.	3 departments (IT, Accounts and Sales)
3.	Each department is required to be in different VLANs.
4.	Each department is required to have a wireless network for the users.
5.	Host devices in the network are required to obtain IPv4 addresses automatically.
6.	Devices in all departments are required to communicate with each other.

---

# Network Topology

<img width="1598" height="688" alt="Topology_diagram" src="https://github.com/user-attachments/assets/97744afc-a38a-47cd-b8c4-329658437f5c" />

---

# IP Addressing Plan

### ISP Assigned Network

| Network        | Subnet Mask   |
| -------------- | ------------- |
| 192.168.4.0/24 | 255.255.255.0 |

### VLAN and Subnet Allocation

| Department | VLAN | Network          | Subnet Mask     | Default Gateway |
| ---------- | ---- | ---------------- | --------------- | --------------- |
| IT         | 10   | 192.168.4.0/26   | 255.255.255.192 | 192.168.4.1     |
| Accounts   | 20   | 192.168.4.64/26  | 255.255.255.192 | 192.168.4.65    |
| Sales      | 30   | 192.168.4.128/26 | 255.255.255.192 | 192.168.4.129   |

---

# Implementation


## Step 1: Create VLANs

Created VLANs for the IT, Accounts, and Sales departments.

```terminal
enable
configure terminal

vlan 10
 name IT

vlan 20
 name Accounts

vlan 30
 name Sales
```
---


## Step 2: Assign Access Ports

Assign switch ports to their departmental VLANs.
```terminal
configure terminal

interface range fa0/2-4
 switchport mode access
 switchport access vlan 10

interface range fa0/5-7
 switchport mode access
 switchport access vlan 20

interface range fa0/8-10
 switchport mode access
 switchport access vlan 30
```

### Port Allocation

| Ports          | VLAN    | Department |
| -------------- | ------- | ---------- |
| Fa0/2 - Fa0/4  | VLAN 10 | IT         |
| Fa0/5 - Fa0/7  | VLAN 20 | Accounts   |
| Fa0/8 - Fa0/10 | VLAN 30 | Sales      |

---


## Step 3: Configure the Trunk Link

Configured the uplink between the switch and router as an IEEE 802.1Q trunk.
```terminal
configure terminal

interface fa0/1
 switchport mode trunk
 switchport trunk native vlan 199
```
---


## Step 4: Verify Switch Configuration

Verify VLAN creation and trunk operation.

```verification command
show vlan brief
```
<img width="804" height="328" alt="image" src="https://github.com/user-attachments/assets/9e24da36-4412-45bc-8d40-fe605cf44560" />


```verification command
show interfaces trunk
```
<img width="698" height="235" alt="image" src="https://github.com/user-attachments/assets/023776b8-e1c5-426a-9bfa-2827ddc1f89a" />


Expected results:

* VLANs 10, 20, and 30 are present.
* Access ports are assigned correctly.
* Fa0/1 is operating as a trunk.
  
---

# Router-on-a-Stick Configuration


## Step 5: Enable Router Interface

```terminal
configure terminal

interface g0/0
 no shutdown
```

---


## Step 6: Configure Subinterfaces

Configured router subinterfaces to provide inter-VLAN routing.
```terminal
configure terminal

interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.4.1 255.255.255.192

interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.4.65 255.255.255.192

interface g0/0.30
 encapsulation dot1Q 30
 ip address 192.168.4.129 255.255.255.192
```

### Gateway Assignment

| VLAN    | Gateway       |
| ------- | ------------- |
| VLAN 10 | 192.168.4.1   |
| VLAN 20 | 192.168.4.65  |
| VLAN 30 | 192.168.4.129 |

---


## Step 7: Verify Router Configuration

```verification command
show ip interface brief
```
<img width="817" height="173" alt="image" src="https://github.com/user-attachments/assets/1d85ef81-ed2c-4b6e-89af-15367358ef77" />


Expected results:

* G0/0 is Up/Up
* G0/0.10 is Up/Up
* G0/0.20 is Up/Up
* G0/0.30 is Up/Up

---

# DHCP Configuration


## Step 8: Exclude Reserved Addresses

Reserved the first available addresses in each subnet for special devices such as printers, servers, and access points.
```terminal
ip dhcp excluded-address 192.168.4.1 192.168.4.10

ip dhcp excluded-address 192.168.4.65 192.168.4.75

ip dhcp excluded-address 192.168.4.129 192.168.4.139
```

---


## Step 9: Create DHCP Pools

### IT Department
```terminal
ip dhcp pool IT
network 192.168.4.0 255.255.255.192
 default-router 192.168.4.1
 dns-server 192.168.4.1
```

### Accounts Department
```terminal
ip dhcp pool Accounts
network 192.168.4.64 255.255.255.192
 default-router 192.168.4.65
 dns-server 192.168.4.65
```

### Sales Department
```terminal
ip dhcp pool Sales
network 192.168.4.128 255.255.255.192
 default-router 192.168.4.129
 dns-server 192.168.4.129
```

---


## Step 10: Configure End Devices for DHCP

On each PC select:

(1)Desktop → (2)IP Configuration → (3)DHCP
<img width="1921" height="1021" alt="image" src="https://github.com/user-attachments/assets/69e79c75-4e87-4e56-8c2f-6f7421653135" />
<img width="1920" height="982" alt="image" src="https://github.com/user-attachments/assets/263d4e80-b024-404f-a4c5-ae4779cca8d6" />
<img width="1920" height="982" alt="image" src="https://github.com/user-attachments/assets/45d40d69-a0f5-4c7e-a15f-a0de021be616" />

Verify that each device automatically receives:

* IP Address
* Subnet Mask
* Default Gateway
* DNS Server
<img width="1921" height="1021" alt="image" src="https://github.com/user-attachments/assets/6ee5da0b-ce07-4201-8e43-f40393f274ad" />

---

# Wireless Network Configuration


## Step 11: Configure Wireless Access Points

For each access point navigate to:
(1)Config → (2)Port 1
<img width="1920" height="982" alt="image" src="https://github.com/user-attachments/assets/991bfdff-8d0b-4697-bf31-3b9e0b8010d7" />
<img width="1920" height="982" alt="image" src="https://github.com/user-attachments/assets/16806fcf-1bf3-4ca4-9ad6-6bd00ea42322" />

Configure:
1. SSID(aka Wifi name)
2. WPA2-PSK Authentication
3. Strong Passphrase(aka Wifi password)
4. AES Encryption
<img width="1921" height="1021" alt="image" src="https://github.com/user-attachments/assets/b28c8f6d-3eb8-4e00-afa2-a0037599b7a5" />


## Step 12: Connect Wireless Clients

### Install Wireless Adapter

1. Power off the laptop.
 <img width="1910" height="1018" alt="image" src="https://github.com/user-attachments/assets/e3bfe35d-72ae-4ed9-b30d-1524122781e4" />
2. Remove the existing module.
   <img width="1918" height="1012" alt="image" src="https://github.com/user-attachments/assets/e03bf721-0e3f-418c-aa2e-c8bf18e6f7ef" />
3. Install the WPC300N wireless adapter.
<img width="1921" height="1021" alt="image" src="https://github.com/user-attachments/assets/c2a34b9e-a5e8-4631-8176-12fa0fe9c7a7" />
4. Power on the laptop.
<img width="1919" height="1016" alt="image" src="https://github.com/user-attachments/assets/1321e225-6f9e-4f2a-aa97-9f03f843eb15" />

### Connect to Wi-Fi

For each laptop navigate to:
(1)Desktop → (2)PC Wireless
<img width="1917" height="1014" alt="image" src="https://github.com/user-attachments/assets/e19e8a1d-03ec-4c90-971f-fde0a37816e8" />
<img width="1921" height="1021" alt="image" src="https://github.com/user-attachments/assets/802d15dd-b949-4a56-8812-3a3c89634951" />

Then:
1. Click Connect.
<img width="1920" height="982" alt="image" src="https://github.com/user-attachments/assets/df325c21-3b3f-464a-aa6d-7a8d00a3599d" />
2. Click Refresh
<img width="1920" height="982" alt="image" src="https://github.com/user-attachments/assets/4c2f1dcf-1657-449f-88ad-2ca33ca30f95" />
3. Select the appropriate SSID for your department.
<img width="1920" height="982" alt="image" src="https://github.com/user-attachments/assets/7fb61276-28b9-4a67-93e1-3843cb58138d" />
4. Click Connect.
 <img width="1920" height="982" alt="image" src="https://github.com/user-attachments/assets/e0e8b37c-c33b-40dc-b9db-09deeaeeac42" />
5. Enter the wireless password.
<img width="1920" height="982" alt="image" src="https://github.com/user-attachments/assets/81f4e406-d072-4cf4-b03f-2426cb664e96" />

---

## Verify Inter-VLAN connectivity by using ping command to check connectivity between devices in different VLANs

 ```Ping command use case in Command Prompt
ping device_ip_address
<img width="630" height="283" alt="image" src="https://github.com/user-attachments/assets/51838331-7941-4727-a268-827b5e2b6fca" />
```

# Successfully implemented:

✅ VLAN Segmentation

✅ Inter-VLAN Routing

✅ DHCP Services

✅ Wireless Networking

✅ Departmental Network Isolation

✅ End-to-End Connectivity

All devices successfully obtained IP addresses automatically and communicated across VLAN boundaries through Router-on-a-Stick routing.

---

# Skills Demonstrated

* Network Design & Planning
* Cisco Switch Configuration
* Cisco Router Configuration
* VLAN Implementation
* 802.1Q Trunking
* Router-on-a-Stick
* DHCP Deployment
* Wireless Networking
* IPv4 Addressing
* Subnetting
* Connectivity Testing

---

# Future Improvements

Potential enhancements include:

* Internet Connectivity
* NAT/PAT Configuration
* OSPF Routing
* Layer 3 Switching

---

# Project Files

| File                      | Description                 |
| ------------------------- | --------------------------- |
| Simon_Enterprises_Lab.pkt | Cisco Packet Tracer Project |
| Topology_Diagram.png      | Network Topology image      |
| README.md                 | Project Documentation       |

---

# Author

**Nyasha Prince Jiri**

IT Support Technician | Network Engineer 

*Helping People One Ping At A Time* 🚀

---

## Watch me build this lab on YouTube => 
