# SSH Brute Force Detection Lab (Ubuntu + Kali)

---

## Objective

Simulate an SSH brute-force attack in an isolated lab environment and detect the attack using system logs and packet-level analysis.

---

## Lab Environment

- Target System: Ubuntu 24.04 (Host Machine)

- Attacker System: Kali Linux (VirtualBox VM)

- Network: Host-Only Adapter (192.168.56.0/24)

- Service Tested: OpenSSH (Port 22)

---

## Attack Simulation

Multiple SSH login attempts were initiated from Kali:

``` ssh ashborn@192.168.56.1 ```

Incorrect passwords were entered repeatedly to simulate brute-force behavior.

---

## Log-Based Detection

Authentication logs were analyzed using:

``` sudo grep -a "Failed password" /var/log/auth.log ```

Findings:

- Attacker IP: 192.168.56.102

- Total failed attempts: 30

- Attack duration: ~30 seconds

- Pattern: 3 failed attempts per SSH session

---

## Packet-Level Correlation (Wireshark)

Traffic was captured on vboxnet0 interface and filtered using:

``` tcp.port == 22 ```

Observed behavior:

- Multiple TCP sessions from attacker IP

- Unique ephemeral source port per session

- 3 authentication attempts per TCP session

- Connection termination after failed attempts

- New session initiated with new source port

This confirms brute-force behavior at both application and network layers.

---

## Mitigation Recommendations

- Implement Fail2Ban to block repeated authentication failures

- Enforce SSH key-based authentication

- Restrict SSH access using firewall rules (UFW)

- Monitor authentication logs regularly

---

## Skills Demonstrated

- Linux log analysis

- Pattern-based log parsing (grep, awk)

- SSH protocol understanding

- TCP handshake analysis

- Packet capture with Wireshark

- Correlation between application and network layers
