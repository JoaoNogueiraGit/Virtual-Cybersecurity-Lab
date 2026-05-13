# 🏗️ Architecture and Network Setup



This document details the hardware specifications, network segregation, and firewall routing rules implemented in this virtualized SOC environment.



## 1. Virtual Machine Specifications



The lab was built using Oracle VirtualBox. To ensure smooth performance while simulating an enterprise environment, the following resources were allocated:



* **pfSense (Firewall/Router):** 1 CPU Core | 2GB RAM | 3 Network Adapters

* **Kali Linux (Attacker/LAN Zone):** 2 CPU Cores | 4GB RAM | 1 Network Adapter

* **Ubuntu Server 24.04 (Target/DMZ Zone):** 1 CPU Core | 1GB RAM | 1 Network Adapter



## 2. Network Topology & VirtualBox Interfaces



To isolate the lab from the host machine and ensure controlled traffic flow, VirtualBox network adapters were configured as follows:



* **WAN (Interface `em0`):** Configured as `NAT` (or `Bridged`). This provides upstream internet access to the pfSense firewall to download packages (like Suricata).

* **LAN (Interface `em1` - `192.168.10.1/24`):** Configured as `Internal Network` (Name: `lan\_net`). This represents the internal corporate network, hosting the Kali Linux machine (`192.168.10.10`).

* **DMZ (Interface `em2` - `192.168.20.1/24`):** Configured as `Internal Network` (Name: `dmz\_net`). This represents the Demilitarized Zone, hosting the vulnerable Ubuntu Server (`192.168.20.10`).



## 3. pfSense Firewall Rules (Stateful Routing)



A core objective of this lab is enforcing the \*\*Principle of Least Privilege\*\*. The pfSense firewall was configured with strict stateful rules to prevent unauthorized lateral movement:



### LAN Rules

* **Action:** `PASS`

* **Source:** `LAN net`

* **Destination:** `ANY`

* **Description:** The LAN network (where the SOC Analyst / Admin resides) is allowed to initiate traffic to the Internet and into the DMZ.



### DMZ (OPT1) Rules

* **Action:** `BLOCK`

* **Source:** `OPT1 net`

* **Destination:** `LAN net`

* **Description:** **Critical Security Rule.** Prevents any system compromised within the DMZ (e.g., the Ubuntu Web Server) from pivoting and initiating a connection into the internal LAN. Traffic is only allowed if it is a response to a connection established by the LAN (handled automatically by pfSense's state table).



\*\[Optional: Insert a screenshot of your pfSense firewall rules here]\*

