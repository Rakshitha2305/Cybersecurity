# SSH Login Monitoring, Detection & Incident Response with Wazuh

## Overview

A practical blue-team/SOC lab that monitors SSH authentication activity on a Debian Linux server using Wazuh.

The project detects repeated failed SSH login attempts, identifies brute-force behavior, automatically blocks the suspicious source IP using Wazuh Active Response, verifies that the attacker can no longer connect, and automatically removes the firewall block after a configured timeout.

### Workflow

```text
SSH Login Attempts
        ↓
Journald Log Collection
        ↓
Wazuh Detection
        ↓
Brute-Force Alert
        ↓
Automatic Firewall Blocking
        ↓
SSH Connection Verification
        ↓
Automatic Unblocking
```

---

## Objectives

- Monitor SSH authentication logs on Linux.
    
- Collect journald events using the Wazuh agent.
    
- Detect repeated failed SSH login attempts.
    
- Create custom Wazuh detection rules.
    
- Map brute-force activity to MITRE ATT&CK.
    
- Automatically block suspicious IP addresses.
    
- Verify firewall-based containment.
    
- Automatically remove the block after a timeout.
    
- Monitor successful SSH logins and session activity.
    

---

## Lab Architecture

```text
┌─────────────────────────────┐
│ Kali Linux                  │
│                             │
│ Wazuh Manager               │
│ Wazuh Dashboard             │
│ OpenSearch Indexer          │
│ Attack Simulation           │
└──────────────┬──────────────┘
               │ SSH
               ▼
┌─────────────────────────────┐
│ Debian Linux                │
│                             │
│ OpenSSH Server              │
│ Wazuh Agent                 │
│ systemd-journald            │
│ iptables Firewall           │
└─────────────────────────────┘
```

### Lab Systems

|System|IP Address|Role|
|---|---|---|
|Kali Linux|`172.20.0.214`|Attack simulation and Wazuh monitoring|
|Debian Linux|`172.20.1.226`|Monitored SSH server|

---

## Technologies Used

- Wazuh Manager
    
- Wazuh Agent
    
- Wazuh Dashboard
    
- OpenSearch
    
- Kali Linux
    
- Debian Linux
    
- OpenSSH
    
- systemd-journald
    
- iptables
    
- MITRE ATT&CK
    

---

## Detection Rules

Two custom Wazuh rules were created in:

```text
/var/ossec/etc/rules/local_rules.xml
```

|Rule ID|Detection Logic|Level|
|---|---|---|
|`100100`|Five failed attempts using a non-existent username from the same IP within 120 seconds|10|
|`100101`|Five failed password attempts from the same IP within 120 seconds|10|

Both rules use:

```text
MITRE ATT&CK: T1110 - Brute Force
```

The rules correlate multiple authentication failures using the same source IP address.

---

## Active Response

Wazuh Active Response was configured to execute `firewall-drop` when either custom rule is triggered.

```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>100100,100101</rules_id>
  <timeout>600</timeout>
</active-response>
```

### Response Behavior

- Suspicious IP is added to the Debian firewall.
    
- Incoming traffic from that IP is dropped.
    
- SSH connections from the blocked IP time out.
    
- The firewall rule is automatically removed after 600 seconds.
    

```text
600 seconds = 10 minutes
```

---

## Testing Performed

### 1. Invalid Username Brute Force

Five SSH attempts were generated using a non-existent username.

Result:

```text
Rule 100100 triggered
```

### 2. Incorrect Password Brute Force

Five failed password attempts were generated using an existing username.

Result:

```text
Rule 100101 triggered
```

### 3. Successful SSH Login

A valid SSH login was performed and Wazuh generated events for:

- Successful authentication — Rule `5715`
    
- Session opened — Rule `5501`
    
- Session closed — Rule `5502`
    

---

## Detection → Containment → Recovery

### Detection

Wazuh detected five failed SSH attempts from the same source IP within 120 seconds.

```text
Rule: 100100
Severity: Level 10
Technique: T1110 - Brute Force
Source IP: 172.20.0.214
```

### Containment

Wazuh Active Response executed `firewall-drop`.

```text
Rule: 651
Action: add
```

The Debian firewall showed:

```text
DROP all -- 172.20.0.214 0.0.0.0/0
```

A new SSH connection from Kali timed out, confirming that the source IP was blocked.

### Recovery

After the 600-second timeout, Wazuh automatically removed the firewall rule.

```text
Rule: 652
Action: delete
```

The block lifecycle was:

```text
Brute-force detected
        ↓
IP blocked
        ↓
SSH connection timed out
        ↓
600-second timeout
        ↓
IP unblocked
        ↓
Firewall rule removed
```

---

## Evidence

### Brute-Force Detection

![Wazuh brute-force alert](https://chatgpt.com/c/screenshots/brute-force-alert.png)

### Active Response Block

![Wazuh active response block](https://chatgpt.com/c/screenshots/active-response-blocked.png)

### Firewall DROP Rule

![Debian firewall DROP rule](https://chatgpt.com/c/screenshots/firewall-drop-rule.png)

### Blocked SSH Connection

![SSH connection timeout](https://chatgpt.com/c/screenshots/ssh-timeout.png)

### Automatic Unblocking

![Wazuh active response unblock](https://chatgpt.com/c/screenshots/active-response-unblocked.png)

---

## Results

|Test|Result|
|---|---|
|Journald monitoring|Successful|
|Invalid-user brute-force detection|Successful|
|Wrong-password brute-force detection|Successful|
|Custom rule `100100`|Triggered|
|Custom rule `100101`|Triggered|
|MITRE ATT&CK mapping|`T1110`|
|Active Response blocking|Successful|
|Firewall DROP rule creation|Verified|
|SSH connection after blocking|Timed out|
|Automatic unblocking|Successful|
|Firewall rule removal|Verified|
|Successful login detection|Successful|

---

## Skills Demonstrated

- Linux security monitoring
    
- SSH authentication analysis
    
- Journald log collection
    
- Wazuh rule creation
    
- Event correlation
    
- Brute-force detection
    
- MITRE ATT&CK mapping
    
- SOC alert investigation
    
- Active Response configuration
    
- Firewall-based containment
    
- Incident-response automation
    
- Detection validation
    
- Security evidence collection
    

---

## Limitations

- Detection currently focuses on repeated attempts from the same source IP.
    
- Distributed brute-force attacks using multiple IP addresses require additional detection logic.
    
- IP blocking may affect legitimate users if a shared IP is blocked.
    
- The current temporary block duration is 600 seconds.
    
- The project was tested in a controlled Kali-to-Debian lab environment.
    

---

## Future Improvements

- Detect successful login immediately after multiple failed attempts.
    
- Add trusted-IP allowlisting.
    
- Add email or messaging notifications.
    
- Add repeated-offender escalation.
    
- Add GeoIP enrichment.
    
- Create a custom Wazuh dashboard for SSH activity.
    
- Integrate incident ticket creation.
    
- Detect suspicious commands after successful login.
    

---

## Project Status

**Completed**

The project successfully demonstrates an end-to-end SOC workflow:

```text
Detect → Contain → Recover
```

Detection, automated firewall blocking, SSH connection verification and automatic recovery were tested successfully.