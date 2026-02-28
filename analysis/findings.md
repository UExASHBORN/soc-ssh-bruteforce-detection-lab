# Technical Findings – SSH Brute Force Simulation

---

## 1. Source IP Identification

From /var/log/auth.log:

Failed password for ashborn from 192.168.56.102 port XXXXX ssh2

The attacker IP was consistently:

```
192.168.56.102
```

This IP corresponds to the Kali Linux VM in the Host-Only network (192.168.56.0/24).

---

## 2. Total Failed Attempts

Command used:

```
sudo grep -a "Failed password" /var/log/auth.log | wc -l
```

Observed failed attempts: 30

This confirms repeated authentication attempts in short time window.

---

## 3. Three Attempts Per SSH Session

From Wireshark capture:

Filter used:

```
tcp.port == 22
```

Observed behavior:

- TCP 3-way handshake (SYN → SYN-ACK → ACK)

- SSH protocol negotiation

- 3 authentication attempts

- Connection terminated (FIN)

- New TCP session started with new source port

This indicates SSH allows multiple authentication retries per TCP session before closing connection.

---

## 4. Ephemeral Source Port Behavior

Each new SSH connection from attacker used a different source port:

Examples observed:

- 33684

- 39494

- 40920

- 44792

- 50080

- 51266

- 53272

- 54948

Each new SSH session used a different ephemeral source port, indicating repeated connection attempts rather than a single persistent session.

Destination port remained constant:

- 22 (SSH)

---

## 5. Attack Time Window

- Log timestamps show all attempts occurred within approximately 30 seconds.

- This short burst of repeated authentication attempts matches brute-force characteristics.

---

## 6. Layer Correlation

The attack was verified at two layers:

Application Layer:

- Authentication failure entries in /var/log/auth.log

Network Layer:

- Multiple TCP sessions to port 22

- Repeated session establishment and termination

- Pattern of 3 failed attempts per session

Log and packet analysis aligned, confirming brute-force characteristics.

---

## 7. Security Implications

If exposed to internet:

- System would be vulnerable to automated password brute-force

- Without automated response or rate limiting, attacker attempts would continue until manual intervention.

---

## 8. Additional Hardening Considerations (Not Implemented in This Lab)

- Disable password authentication (PasswordAuthentication no)

- Use SSH key-based authentication

- Restrict SSH access via firewall (UFW / iptables)

- Monitor logs with centralized logging (SIEM)

---

## 9. Defensive Implementation – Fail2Ban Mitigation

Objective

Implement automated blocking for repeated SSH authentication failures.

Configuration
```
sudo apt install fail2ban
sudo nano /etc/fail2ban/jail.local
```

Configuration used:
```
[sshd]
enabled = true
maxretry = 3
findtime = 60
bantime = 300
```

Validation Test

1.Initiated 3 failed SSH login attempts from Kali

2.Checked jail status:

```
sudo fail2ban-client status sshd
```

Result:

- Banned IP:```192.168.56.102```

- Ban duration: 300 seconds

- Security Impact

- Prevents repeated brute-force attempts

- Reduces attack window

- Converts detection into automated response

