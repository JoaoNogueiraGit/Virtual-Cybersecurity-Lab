# 🗡️ Red Team Attack Simulations & Blue Team Detection

This document details the offensive security simulations executed from the Kali Linux machine (LAN Network) and the corresponding defensive logging and detection captured by the pfSense firewall and Suricata NIDS.

## Simulation 1: Network Reconnaissance (Port Scanning)

* **Objective:** To identify active services on the DMZ target and observe how the IDS handles and logs aggressive vs. stealth network scanning.
* **Target Setup:** Ubuntu Server (`192.168.20.10`) with basic services enabled.
* **Attack Vector (Red Team):** From the Kali Linux machine (`192.168.10.10`), an Aggressive Nmap scan and a Stealth SYN scan were launched to map the target:
  ```bash
  # Aggressive Scan (OS detection, version detection, script scanning)
  nmap -A 192.168.20.10
  
  # Stealth SYN Scan (Half-open connections)
  nmap -sS -p- 192.168.20.10
  ```
* **Detection & Logs (Blue Team):** The pfSense firewall correctly allowed the traffic based on the stateful rules (LAN to DMZ is permitted). However, the Suricata NIDS engine inspected the
 traffic flow and generated alerts for the aggressive scanning behavior, matching signatures like `ET SCAN Nmap User-Agent` or specific protocol scanning anomalies.

---

## Simulation 2: Deep Packet Inspection (Malware Signature Detection)

* **Objective:** To prove that Suricata is not just looking at IP addresses and ports, but actively opening packets and reading their contents (DPI) to detect malicious payloads or Command & Control (C2) traffic.
* **Target Setup:** To create a valid HTTP flow, a temporary web server was spawned on the Ubuntu Server (DMZ) using Python:
  ```bash
  sudo python3 -m http.server 80
  ```
* **Attack Vector:** A crafted HTTP request was sent from the Kali machine to the target. The request contained a known malware signature ("BlackSun") hidden inside the User-Agent string:
  ```bash
  curl -A "BlackSun" [http://192.168.20.10](http://192.168.20.10)
  ```
* **Detection & Logs:** This simulation was a complete success. Suricata intercepted the HTTP packet, analyzed the payload, and triggered two distinct alerts in the pfSense dashboard:
  1. **`ET INFO Python SimpleHTTP ServerBanner`**: The official Emerging Threats rules correctly identified that a temporary Python web server was unexpectedly spun up in the DMZ.
  2. **`CUSTOM ALERT: BlackSun Trojan Attack Detected!`**: The custom DPI rule (see Suricata IDS implementation.md) successfully matched the string inside the HTTP header and fired the critical alert.
