# SSH Brute Force Detection Lab (Ubuntu + Kali)

---

## Blue Team Workflow Demonstrated:
Recon → Detection → Analysis → Mitigation → Validation

---

## Objective

- Simulate an SSH brute-force attack in an isolated lab environment and detect the attack using system logs and packet-level analysis.

- Result: Automated IP blocking triggered after 3 failed SSH authentication attempts. Verified through auth.log evidence and attacker-side connection refusal.

---

## Lab Environment

- Target System: Ubuntu 24.04 (Host Machine)

- Attacker System: Kali Linux (VirtualBox VM)

- Network: Host-Only Adapter (192.168.56.0/24)

- Service Tested: OpenSSH (Port 22)

---

## Attack Simulation

Multiple SSH login attempts were initiated from Kali:

```
 ssh ashborn@192.168.56.1
```

Incorrect passwords were entered repeatedly to simulate brute-force behavior.

---

## Log-Based Detection

Authentication logs were analyzed using:

``` 
sudo grep -a "Failed password" /var/log/auth.log
```

Findings:

- Attacker IP: 192.168.56.102

- Total failed attempts: 30

- Attack duration: ~30 seconds

- Pattern: 3 failed attempts per SSH session

---

## Packet-Level Correlation (Wireshark)

Traffic was captured on vboxnet0 interface and filtered using:

```
tcp.port == 22
```

Observed behavior:

- Multiple TCP sessions from attacker IP

- Unique ephemeral source port per session

- 3 authentication attempts per TCP session

- Connection termination after failed attempts

- New session initiated with new source port

---

## Skills Demonstrated

- Linux log analysis

- Pattern-based log parsing (grep, awk)

- SSH protocol understanding

- TCP handshake analysis

- Packet capture with Wireshark

- Correlation between application and network layers

---

## Defensive Validation

Fail2Ban was implemented to automatically block repeated SSH authentication failures.

After 3 failed login attempts:

- Attacker IP was banned

- SSH access was blocked

- Connection refused observed from attacker system

See detailed implementation and evidence in:

```
analysis/findings.md
```
