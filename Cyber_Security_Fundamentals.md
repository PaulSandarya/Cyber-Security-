#  Fundamentals

## 1. Networking fundamentals

### 1.1 OSI model vs TCP/IP model

| OSI layer | Function | Examples / PDU |
| --- | --- | --- |
| 7 Application | User-facing network services | HTTP, DNS, SMTP, FTP |
| 6 Presentation | Encoding, encryption, compression | TLS, JPEG, ASCII |
| 5 Session | Establishing and managing sessions | NetBIOS, RPC |
| 4 Transport | End-to-end delivery, ports | TCP, UDP (segment/datagram) |
| 3 Network | Logical addressing, routing | IP, ICMP (packet) |
| 2 Data link | Local delivery using MAC addresses | Ethernet, ARP, switches (frame) |
| 1 Physical | Bits over the medium | Cables, Wi-Fi radio |

TCP/IP collapses this into four layers: Application (OSI 5 to 7), Transport, Internet, and Network Access (OSI 1 to 2). Memory aid: "Please Do Not Throw Sausage Pizza Away" (Physical to Application).

### 1.2 Key protocols to know

- **TCP** – connection-oriented and reliable; uses the three-way handshake (SYN, SYN-ACK, ACK) and retransmits lost data
- **UDP** – connectionless, faster, no delivery guarantee; used by DNS queries, streaming, VoIP, DHCP
- **ARP** – maps IP addresses to MAC addresses on a local network; vulnerable to spoofing/poisoning
- **ICMP** – diagnostics and errors (ping, traceroute); often abused for tunnelling and recon
- **DNS** – translates names to IP addresses; record types A, AAAA, CNAME, MX, TXT, NS, PTR
- **DHCP** – automatically assigns IP, gateway and DNS (DORA: Discover, Offer, Request, Acknowledge)
- **HTTP/HTTPS** – web protocol; HTTPS = HTTP over TLS

### 1.3 Common ports

| Port | Service | Port | Service |
| --- | --- | --- | --- |
| 20/21 | FTP (data/control) | 110 / 995 | POP3 / POP3S |
| 22 | SSH / SFTP | 143 / 993 | IMAP / IMAPS |
| 23 | Telnet (insecure) | 161/162 | SNMP |
| 25 / 587 | SMTP / SMTP submission | 389 / 636 | LDAP / LDAPS |
| 53 | DNS (UDP/TCP) | 445 | SMB |
| 67/68 | DHCP | 1433 / 3306 | MSSQL / MySQL |
| 80 / 443 | HTTP / HTTPS | 3389 | RDP |

### 1.4 TLS handshake in brief (TLS 1.2 vs 1.3)

- **Client Hello** – client sends supported TLS versions, cipher suites and a random value
- **Server Hello and certificate** – server picks the cipher suite and presents its certificate
- **Certificate validation** – client checks the chain of trust, expiry and hostname
- **Key exchange** – both sides derive a shared session key (ephemeral Diffie-Hellman gives forward secrecy)
- **Encrypted traffic** – data is protected with a symmetric cipher such as AES-GCM
- **TLS 1.3** – faster (one round trip), removes weak ciphers and static RSA key exchange

### 1.5 Subnetting quick reference

| CIDR | Subnet mask | Usable hosts |
| --- | --- | --- |
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /26 | 255.255.255.192 | 62 |
| /27 | 255.255.255.224 | 30 |
| /28 | 255.255.255.240 | 14 |
| /30 | 255.255.255.252 | 2 |

Formula: usable hosts = 2^(32 minus prefix) minus 2. Private ranges (RFC 1918): 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16.

## 2. Linux fundamentals

### 2.1 Commands you must know

```text
ls -la                  # list files with permissions and hidden files
cat, less, head, tail -f   # view files / follow a log live
grep -ri "failed" /var/log # search text recursively
find / -perm -4000 2>/dev/null   # find SUID binaries
ps aux | grep sshd      # list processes
ss -tulpn  (netstat -tulpn)      # listening ports and owning process
lsof -i :22             # who is using port 22
chmod 640 file; chown user:group file
sudo -l                 # what can this user run as root
journalctl -u ssh --since "1 hour ago"
last, lastb, w, who     # login history and failed logins
crontab -l              # scheduled tasks (common persistence spot)
```

### 2.2 File permissions

Permissions are shown as rwx for owner, group and others. Numeric values: r=4, w=2, x=1. So chmod 754 means owner rwx (7), group r-x (5), others r-- (4). Special bits: SUID (4000) runs a file as its owner, SGID (2000) runs as its group, sticky bit (1000) stops users deleting others' files in a shared directory such as /tmp.

### 2.3 Important Linux files and logs

- **/etc/passwd and /etc/shadow** – user accounts and password hashes (shadow is root-readable only)
- **/var/log/auth.log (Debian) or /var/log/secure (RHEL)** – authentication events, sudo, SSH logins
- **/var/log/syslog or /var/log/messages** – general system logs
- **~/.bash_history** – command history, useful in investigations
- **/etc/crontab and /etc/cron.*** – scheduled jobs
- **~/.ssh/authorized_keys** – keys allowed to log in (attackers add their own for persistence)

## 3. Windows and Active Directory

### 3.1 Core concepts

- **Active Directory (AD)** – Microsoft directory service that stores users, groups, computers and policies
- **Domain Controller (DC)** – server that hosts AD and authenticates users
- **Group Policy (GPO)** – centrally enforces settings on domain-joined machines
- **Kerberos** – default domain authentication protocol using tickets (TGT and service tickets)
- **NTLM** – older challenge-response authentication; weaker and prone to relay and pass-the-hash
- **Registry** – configuration database; Run keys are a common persistence location

### 3.2 Windows Security event IDs worth memorising

| Event ID | Meaning | Why it matters |
| --- | --- | --- |
| 4624 | Successful logon | Check logon type (2 interactive, 3 network, 10 RDP) |
| 4625 | Failed logon | Many in a row suggests brute force or password spraying |
| 4634 / 4647 | Logoff | Session timeline |
| 4648 | Logon with explicit credentials | Possible lateral movement |
| 4672 | Special privileges assigned | Admin-level logon |
| 4688 | New process created | Needs command-line auditing; key for hunting |
| 4720 / 4726 | User account created / deleted | Persistence via new accounts |
| 4732 | Member added to a local security group | Privilege escalation |
| 4768 / 4769 | Kerberos TGT / service ticket requested | Kerberoasting detection |
| 1102 | Audit log cleared | Strong sign of attacker covering tracks |

## 4. Security principles and terminology

- **CIA triad** – Confidentiality (only authorised access), Integrity (data not altered improperly), Availability (systems usable when needed)
- **AAA** – Authentication (who are you), Authorization (what can you do), Accounting (what did you do)
- **Vulnerability** – a weakness in a system or process
- **Threat** – something that can exploit a vulnerability (attacker, malware, natural event)
- **Risk** – likelihood multiplied by impact of a threat exploiting a vulnerability
- **Exploit** – code or technique that takes advantage of a vulnerability
- **Zero-day** – a vulnerability with no patch available yet
- **Least privilege** – users and systems get only the access they need
- **Defense in depth** – multiple layered controls so that one failure is not fatal
- **Separation of duties** – no single person controls a whole critical process
- **Non-repudiation** – a party cannot deny an action (for example, digital signatures)

## 5. interview questions and model answers

### Networking

#### Q1. Explain the OSI model and why it matters for security.

**Answer:** The OSI model splits network communication into seven layers: Physical, Data Link, Network, Transport, Session, Presentation and Application.

It matters because it gives a common language for troubleshooting and for placing controls. For example, a firewall filtering by IP works at Layer 3, a stateful firewall at Layer 4, a WAF at Layer 7, and ARP spoofing is a Layer 2 attack.

#### Q2. What is the difference between TCP and UDP?

**Answer:** TCP is connection-oriented, reliable and ordered, using a three-way handshake and acknowledgements. UDP is connectionless and faster but does not guarantee delivery or order.

Security angle: TCP handshakes enable SYN flood attacks and port scanning by connection state; UDP is common in amplification DDoS (DNS, NTP) because source addresses can be spoofed.

#### Q3. Describe the TCP three-way handshake.

**Answer:** The client sends SYN, the server replies SYN-ACK, and the client completes with ACK. After that the connection is established.

In a SYN flood, the attacker sends many SYNs without completing the handshake, exhausting the server's half-open connection table.

#### Q4. What happens when you type a URL into a browser and press Enter?

**Answer:** The browser checks its cache, then resolves the domain through DNS (browser, OS, then recursive resolver, then root, TLD and authoritative servers).

It opens a TCP connection to the IP on port 443, performs the TLS handshake (certificate validation and key exchange), then sends an HTTP request. The server responds, and the browser renders the page and loads additional resources.

Security points to mention: DNS spoofing, certificate validation, MITM protection through HTTPS and HSTS, and cookies/session handling.

#### Q5. What is DNS and how can it be abused by attackers?

**Answer:** DNS translates domain names to IP addresses. Abuse includes:

- DNS spoofing or cache poisoning to redirect users to malicious sites
- DNS tunnelling to exfiltrate data or run command and control over DNS queries
- DGA (domain generation algorithms) used by malware to find C2 servers
- DNS amplification DDoS

Defenses: DNSSEC, DNS filtering, monitoring for long or high-entropy queries, restricting outbound DNS to approved resolvers.

#### Q6. What is ARP and what is ARP poisoning?

**Answer:** ARP maps IP addresses to MAC addresses inside a local network. ARP poisoning is when an attacker sends forged ARP replies so victims associate the attacker's MAC with another host's IP (often the gateway), enabling man-in-the-middle attacks.

Mitigation: dynamic ARP inspection, static ARP entries for critical hosts, port security, network segmentation and encrypted protocols.

#### Q7. What is the difference between a switch, router and firewall?

**Answer:** A switch forwards frames within a LAN using MAC addresses (Layer 2). A router forwards packets between networks using IP addresses (Layer 3). A firewall enforces security policy by allowing or blocking traffic based on rules (IP, port, state, application and so on).

#### Q8. What is NAT and does it provide security?

**Answer:** Network Address Translation maps private internal addresses to a public address. It conserves IPv4 addresses and incidentally hides internal topology, but it is not a security control by itself. A firewall policy is still required.

#### Q9. How does HTTPS protect data, and what does it not protect against?

**Answer:** HTTPS uses TLS to provide encryption (confidentiality), integrity checks and server authentication through certificates.

It does not protect against vulnerabilities in the application itself (SQL injection, XSS), compromised endpoints, phishing sites that have valid certificates, or malicious content served over HTTPS.

### Linux and Windows

#### Q10. How would you find which process is listening on a suspicious port in Linux?

**Answer:** Run ss -tulpn or lsof -i :PORT to see the process and PID, then inspect it with ps aux, check the binary path with ls -l /proc/PID/exe, review its parent process and open files, and compare against baseline. If it is malicious, isolate the host and capture evidence before killing the process.

#### Q11. Explain Linux file permissions. What does chmod 755 mean?

**Answer:** The three digits are owner, group and others. 7 = rwx, 5 = r-x. So 755 gives the owner full access and everyone else read and execute only. It is typical for directories and executables, while sensitive files such as private keys should be 600.

#### Q12. What is SUID and why is it a privilege-escalation risk?

**Answer:** SUID makes a binary run with the file owner's privileges, usually root. If a SUID binary is misconfigured or exploitable (for example, one that can spawn a shell or write arbitrary files), a low-privileged user can gain root. Auditing with find / -perm -4000 is a standard check, and GTFOBins lists abusable binaries.

#### Q13. What is Active Directory and why do attackers target it?

**Answer:** AD is the central identity and policy store for most enterprises. Compromising a domain admin or the domain controller effectively gives control of every domain-joined system. Common attacks include password spraying, Kerberoasting, pass-the-hash, DCSync, and abusing misconfigured delegation or ACLs.

#### Q14. What is the difference between Kerberos and NTLM?

**Answer:** Kerberos is ticket-based, uses a trusted third party (the KDC on the DC) and supports mutual authentication. NTLM is a challenge-response protocol that is weaker, lacks mutual authentication and is vulnerable to relay and pass-the-hash attacks. Best practice is to disable NTLM where possible.

#### Q15. Which Windows event IDs would you look at for suspicious logons?

**Answer:** 4625 (failed logon) for brute force and spraying, 4624 (successful logon, noting logon type and source IP), 4672 (special privileges), 4648 (explicit credentials), 4768/4769 for Kerberos activity, and 1102 for cleared logs.

### Security principles

#### Q16. Explain the CIA triad with an example of each being violated.

**Answer:** Confidentiality: a database leak exposes customer data. Integrity: an attacker alters invoice bank details. Availability: ransomware or DDoS makes a service unusable.

#### Q17. What is the difference between a vulnerability, a threat and a risk?

**Answer:** A vulnerability is a weakness, a threat is something that can exploit it, and risk is the potential for loss when a threat exploits a vulnerability (likelihood times impact). Example: an unpatched server (vulnerability), a ransomware gang (threat), and the business impact of downtime and data loss (risk).

#### Q18. What is defense in depth?

**Answer:** A layered strategy where multiple independent controls protect an asset: for example perimeter firewall, network segmentation, EDR on endpoints, MFA, least privilege, encryption, logging and monitoring, and user training. If one layer fails, others still reduce the impact.

#### Q19. What is the difference between authentication and authorization?

**Answer:** Authentication verifies identity (password, MFA, certificate). Authorization decides what an authenticated identity may access (roles, ACLs, policies). Broken access control is the number one OWASP risk precisely because systems authenticate users but fail to authorize them correctly.

#### Q20. What is multi-factor authentication and what are its weaknesses?

**Answer:** MFA requires two or more factor types: something you know, have or are. Weaknesses: MFA fatigue (push bombing), SIM swapping against SMS codes, real-time phishing proxies (such as Evilginx) that steal session cookies, and weak recovery flows. Phishing-resistant options include FIDO2 security keys and passkeys.


