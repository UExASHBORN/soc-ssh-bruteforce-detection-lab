# Technical Findings – SSH Brute Force Simulation

---

## 1. Source IP Identification

From /var/log/auth.log:

Failed password for ashborn from 192.168.56.102 port XXXXX ssh2

The attacker IP was consistently:

192.168.56.102

This IP corresponds to the Kali Linux VM in the Host-Only network (192.168.56.0/24).

---

## 2. Total Failed Attempts

Command used:

``` sudo grep -a "Failed password" /var/log/auth.log | wc -l ```

Observed failed attempts: 30

This confirms repeated authentication attempts in short time window.

---

## 3. Three Attempts Per SSH Session

From Wireshark capture:

Filter used:

``` tcp.port == 22 ```

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

This is expected TCP behavior.

The client OS assigns a new ephemeral source port for each new outgoing connection.

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

This confirms consistent behavior across log and packet capture evidence.

---

## 7. Security Implications

If exposed to internet:

- System would be vulnerable to automated password brute-force

- Without rate limiting or lockout, attacker could continue attempts indefinitely

---

## 8. Recommended Mitigations

- Deploy Fail2Ban to block repeated failed login attempts

- Disable password authentication (PasswordAuthentication no)

- Use SSH key-based authentication

- Restrict SSH access via firewall (UFW / iptables)

- Monitor logs with centralized logging (SIEM)

