
## 1. Project Overview

This project implements a controlled **Network Intrusion Detection and PCAP Investigation Lab** to demonstrate a practical Blue Team security-monitoring workflow.

The lab uses **Kali Linux** as the attacker machine and **Debian Linux** as the monitored system. Different network reconnaissance activities were generated from Kali, monitored using **Suricata IDS**, captured as PCAP files, and investigated using **Wireshark**.

The project demonstrates the complete workflow:

```text
Reconnaissance → Traffic Capture → IDS Detection → Alert Investigation
                         ↓
                   PCAP Analysis
                         ↓
                    Findings
```

---

## 2. Objectives

The main objectives of the project were:

- Build an attacker–victim network security lab.
    
- Generate different types of network reconnaissance traffic.
    
- Configure Suricata as a Network Intrusion Detection System.
    
- Create custom Suricata detection rules.
    
- Capture network traffic in PCAP format.
    
- Investigate packets using Wireshark.
    
- Correlate IDS alerts with packet-level evidence.
    
- Document the detection and investigation process.
    

---

## 3. Lab Environment

|Component|Details|
|---|---|
|Attacker|Kali Linux|
|Attacker IP|`10.245.144.9`|
|Victim/Monitoring|Debian Linux|
|Victim IP|`10.245.144.188`|
|Monitoring Interface|`enp0s3`|
|IDS|Suricata 7.0.10|
|Scanner|Nmap|
|Packet Analyzer|Wireshark|
|Capture Tool|tcpdump|
|Alert Analysis|jq|

### Network Architecture

```text
              Kali Linux
             10.245.144.9
                 |
                 | Reconnaissance Traffic
                 ↓
             Debian Linux
            10.245.144.188
                 |
            Suricata IDS
              enp0s3
```

---

## 4. Suricata Configuration

Suricata was configured on the Debian machine to monitor the `enp0s3` interface.

### Configuration Files

```text
/etc/suricata/suricata.yaml
/etc/suricata/rules/local.rules
```

The custom rules file was included using:

```yaml
rule-files:
  - suricata.rules
  - /etc/suricata/rules/local.rules
```

The configuration was validated successfully with:

```text
Configuration provided was successfully loaded. Exiting.
```

### Evidence

- `03-suricata-interface-and-rules.png`
    
- `05-suricata-rule-validation.png`
    

---

## 5. Custom IDS Rules

Four custom Suricata rules were created to detect reconnaissance-related traffic.

|SID|Detection|Signature|
|---|---|---|
|`1000001`|TCP SYN reconnaissance|`CUSTOM Nmap SYN Port Scan Detected`|
|`1000002`|ICMP reconnaissance|`CUSTOM ICMP Reconnaissance Detected`|
|`1000003`|TCP connection attempts|`CUSTOM TCP Connection Attempt Burst Detected`|
|`1000004`|UDP reconnaissance|`CUSTOM UDP Reconnaissance Detected`|

The rules were designed specifically for this controlled laboratory environment.

> **Note:** The TCP connection rule is a custom burst detector and is not intended to uniquely identify Nmap's `-sT` scan.

### Evidence

`04-custom-suricata-rules.png`

---

# 6. Reconnaissance and Detection Activities

## 6.1 TCP SYN Scan

The TCP SYN scan was performed from Kali using:

```bash
nmap -sS -p 1-1000 10.245.144.188
```

This generated TCP SYN probes against ports 1–1000.

Suricata detected the activity using:

```text
CUSTOM Nmap SYN Port Scan Detected
```

The traffic was captured in:

```text
nmap-scan.pcap
```

Wireshark analysis identified a large number of SYN probes from:

```text
10.245.144.9 → 10.245.144.188
```

### Evidence

- `06-nmap-syn-scan.png`
    
- `07-suricata-syn-alert.png`
    
- `nmap-scan.pcap`
    

---

## 6.2 ICMP Reconnaissance

ICMP traffic was generated using:

```bash
ping -c 10 10.245.144.188
```

Result:

```text
10 packets transmitted
10 packets received
0% packet loss
```

Suricata detected the repeated ICMP activity using:

```text
CUSTOM ICMP Reconnaissance Detected
```

The traffic was captured in:

```text
icmp-test.pcap
```

Wireshark analysis identified ICMP Echo Requests and Echo Replies between the two systems.

### Evidence

- `10-icmp-ping-test.png`
    
- `11-suricata-icmp-alert.png`
    
- `12-wireshark-icmp-analysis.png`
    
- `icmp-test.pcap`
    

---

## 6.3 TCP Connection Scan

A TCP connection scan was performed using:

```bash
nmap -sT -p 1-1000 10.245.144.188
```

Nmap reported:

```text
1000 closed tcp ports (conn-refused)
```

The traffic was captured in:

```text
tcp-connect-scan.pcap
```

Wireshark was used to investigate the TCP communication between the attacker and target. The investigation did not identify SYN/ACK responses using the specific filter tested, which was consistent with the reported closed ports.

### Evidence

`tcp-connect-scan.pcap`

---

## 6.4 UDP Reconnaissance

UDP reconnaissance was performed using:

```bash
nmap -sU --top-ports 100 10.245.144.188
```

The scan reported:

```text
60 closed udp ports
40 open|filtered udp ports
```

Suricata generated:

```text
CUSTOM UDP Reconnaissance Detected
```

Wireshark was used to investigate UDP traffic from Kali to Debian.

Filter used:

```text
ip.src == 10.245.144.9 && ip.dst == 10.245.144.188 && udp
```

The corresponding traffic was captured in:

```text
udp-scan.pcap
```

### Evidence

- Suricata UDP alert
    
- `udp-scan.pcap`
    
- Wireshark UDP packet analysis
    

---

# 7. PCAP Investigation

Four PCAP files were generated during the experiments:

|PCAP|Purpose|
|---|---|
|`nmap-scan.pcap`|TCP SYN scan investigation|
|`icmp-test.pcap`|ICMP traffic investigation|
|`tcp-connect-scan.pcap`|TCP connection scan investigation|
|`udp-scan.pcap`|UDP reconnaissance investigation|

Wireshark was used to examine:

- Source and destination IP addresses
    
- TCP SYN traffic
    
- TCP responses
    
- UDP probes
    
- ICMP Echo Requests/Replies
    
- Protocol-specific traffic patterns
    

The PCAP files provide independent packet-level evidence supporting the reconnaissance activities and Suricata detections.

---

# 8. Results and Findings

The lab successfully demonstrated detection and investigation of multiple reconnaissance activities.

|Activity|Detection|Investigation Evidence|
|---|---|---|
|TCP SYN scan|Suricata alert|`nmap-scan.pcap`|
|ICMP reconnaissance|Suricata alert|`icmp-test.pcap` + Wireshark|
|TCP connection scan|TCP traffic investigation|`tcp-connect-scan.pcap`|
|UDP reconnaissance|Suricata alert|`udp-scan.pcap` + Wireshark|

The project demonstrated that reconnaissance activity can be identified through a combination of:

```text
Nmap Output
     +
Suricata Alerts
     +
PCAP Evidence
     +
Wireshark Analysis
```

---

# 9. Skills Demonstrated

### Technical Skills

- Linux security environment setup
    
- Network reconnaissance
    
- TCP/IP traffic analysis
    
- Nmap
    
- Suricata IDS
    
- Custom IDS rule writing
    
- Wireshark
    
- PCAP analysis
    
- tcpdump
    
- JSON alert analysis using jq
    

### Blue Team Skills

- Network monitoring
    
- Intrusion detection
    
- Alert investigation
    
- Packet-level investigation
    
- Evidence correlation
    
- Security documentation
    

---

# 10. Limitations

This project was implemented as a controlled learning laboratory.

The custom IDS rules are simple signature-based rules and may not detect sophisticated reconnaissance techniques. They may also require threshold tuning to reduce false positives.

The TCP connection-attempt rule is a custom burst detector rather than a definitive identifier of Nmap `-sT`.

---

# 11. Future Improvements

The lab can be extended with:

- Suricata → Wazuh integration
    
- SIEM-based alert monitoring
    
- Brute-force detection
    
- SSH attack detection
    
- DNS reconnaissance detection
    
- Web attack detection
    
- Malware traffic simulation
    
- MITRE ATT&CK mapping
    
- Automated alert analysis
    
- Centralized logging
    
- Incident-response workflows
    

---

# 12. Conclusion

The project successfully implemented a basic network-security monitoring and investigation workflow.

Reconnaissance traffic was generated from Kali Linux, monitored by Suricata on Debian, detected using custom IDS rules, captured into PCAP files, and investigated using Wireshark.

The lab provided practical experience in:

```text
Network Reconnaissance
        ↓
Traffic Monitoring
        ↓
Intrusion Detection
        ↓
Alert Investigation
        ↓
PCAP Analysis
        ↓
Network Forensics
```

This project demonstrates practical foundational skills relevant to **Blue Team and SOC Analyst activities**, particularly network monitoring, IDS configuration, security-alert investigation, and packet analysis.

---

# 13. Project Evidence

### Screenshots

```text
03-suricata-interface-and-rules.png
04-custom-suricata-rules.png
05-suricata-rule-validation.png
06-nmap-syn-scan.png
07-suricata-syn-alert.png
10-icmp-ping-test.png
11-suricata-icmp-alert.png
12-wireshark-icmp-analysis.png
```

### PCAP Files

```text
nmap-scan.pcap
icmp-test.pcap
tcp-connect-scan.pcap
udp-scan.pcap
```

### Project Repository Structure

```text
Network-Intrusion-Detection-Lab/
│
├── README.md
├── report/
│   └── Network-Intrusion-Detection-PCAP-Investigation.pdf
├── screenshots/
├── rules/
│   └── local.rules
└── pcaps/
    ├── nmap-scan.pcap
    ├── icmp-test.pcap
    ├── tcp-connect-scan.pcap
    └── udp-scan.pcap
```