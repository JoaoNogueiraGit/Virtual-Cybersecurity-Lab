# 🛡️ Suricata NIDS Implementation & Tuning

This document outlines the deployment of the Suricata Network Intrusion Detection System (NIDS) on the pfSense firewall, including the critical troubleshooting steps required to 
make Deep Packet Inspection (DPI) work in a virtualized environment.

## 1. Installation & Global Settings
Suricata was installed via the pfSense Package Manager. The service was bound to the `OPT1` (DMZ) interface to monitor inbound traffic targeting the vulnerable web server.
* **Ruleset:** Enabled the **Emerging Threats (ET) Open Rules** to provide a baseline of industry-standard malware and exploit signatures.
* **Run Mode:** Configured inline/legacy mode depending on the virtual adapter support, ensuring alerts are logged in the pfSense GUI.

## 2. Overcoming Virtualization Challenges (Troubleshooting)
Running an IDS inside a hypervisor introduces specific networking challenges. The following critical configurations were applied to ensure Suricata could actually "see" and inspect the traffic:

### A. Hardware Checksum Offloading (pfSense)
By default, operating systems offload the calculation of network packet checksums to the hardware Network Interface Card (NIC) to save CPU cycles. However, in a virtualized environment (VirtualBox), 
the virtual NIC can miscalculate these, causing pfSense to silently drop the packets before Suricata can inspect them.
* **Fix:** Navigated to `System > Advanced > Networking` in pfSense and checked **Disable hardware checksum offload**.

### B. Promiscuous Mode (VirtualBox)
By default, a virtual network adapter only processes traffic specifically destined for its own MAC address. Since an IDS needs to inspect all traffic passing through the network segment, it must operate in Promiscuous Mode.
* **Fix:** In VirtualBox Network Settings for the pfSense VM, the Promiscuous Mode for the DMZ adapter was changed from **`Deny`** to **`Allow All`**.

## 3. Custom Rule Creation & Variable Tuning
To validate that Deep Packet Inspection was working, custom rules were written. 

### The `$EXTERNAL_NET` Bypass
Standard ET rules often look for attacks originating from `$EXTERNAL_NET` (the internet). Because the Kali Linux attacker machine is located on the internal LAN, Suricata initially ignored its malicious traffic, 
treating it as a "trusted" internal source. To fix this, the `EXTERNAL_NET` variable in Suricata's interface settings was changed from `default` to `any`.

### Custom Malware Signature Rule
To definitively prove the DPI engine was functioning, a custom rule was created to detect a specific string ("BlackSun") inside HTTP traffic over TCP port 80:

```text
alert tcp any any -> any 80 (msg:"CUSTOM ALERT: BlackSun Trojan Attack Detected!"; content:"BlackSun"; sid:1000002; rev:1;)
