# Week 2: Core Security Concepts

> Cryptography, attacks, web security and frameworks, with interview Q&A

[Home](README.md) | [Week 1](Week1_Fundamentals.md) | [Week 2](Week2_Core_Security_Concepts.md) | [Week 3](Week3_Role_Specific_Depth.md) | [Week 4](Week4_Mock_Interviews_and_Polish.md)

---

## How to use this document

Week 2 covers the core security knowledge that appears in almost every interview: cryptography, common attacks, web application security and the major frameworks. Aim for understanding, not memorisation. For each attack, be able to explain how it works, how it is detected and how it is prevented.

| Day | Focus | Hands-on task |
| --- | --- | --- |
| Day 1 | Cryptography: symmetric, asymmetric, hashing, PKI | Use openssl to hash a file, generate an RSA key pair and inspect a certificate |
| Day 2 | Malware, phishing, social engineering, network attacks | Analyse a phishing email header; TryHackMe "Phishing" rooms |
| Day 3 | OWASP Top 10: injection, XSS, access control | PortSwigger labs: SQL injection and XSS (3 labs each) |
| Day 4 | OWASP continued: CSRF, SSRF, auth flaws | PortSwigger labs: CSRF and access control |
| Day 5 | Frameworks: MITRE ATT&CK, Kill Chain, NIST, ISO 27001 | Map one real breach to ATT&CK techniques |
| Day 6 | Revision and Q&A out loud | Answer Q1 to Q25 aloud |
| Day 7 | Rest or catch-up | Revisit weak topics |

## 1. Cryptography essentials

### 1.1 Symmetric vs asymmetric vs hashing

| Type | Keys | Examples | Used for |
| --- | --- | --- | --- |
| Symmetric | One shared secret key | AES-256, ChaCha20 | Bulk data encryption (fast) |
| Asymmetric | Public and private key pair | RSA, ECC, Diffie-Hellman | Key exchange, digital signatures, certificates |
| Hashing | No key; one-way | SHA-256, SHA-3, bcrypt, Argon2 | Integrity checks, password storage |
| Encoding | No key; reversible | Base64, URL encoding | Data formatting, not security |

### 1.2 Important ideas

- **Digital signature** – hash of a message encrypted with the sender's private key; proves integrity, authenticity and non-repudiation
- **PKI** – certificate authorities issue certificates binding a public key to an identity; clients trust a chain leading to a root CA
- **Salting** – adding a unique random value to each password before hashing so identical passwords give different hashes and rainbow tables fail
- **Forward secrecy** – ephemeral session keys mean a later key compromise does not expose past traffic
- **HMAC** – hash combined with a secret key to verify integrity and authenticity
- **Weak or broken algorithms** – MD5, SHA-1, DES, 3DES, RC4; avoid for security purposes
- **Key management** – generation, storage (HSM, KMS), rotation and revocation are usually where real-world crypto fails

```text
# Useful openssl commands
echo -n "hello" | openssl dgst -sha256
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
openssl s_client -connect example.com:443 -showcerts
openssl x509 -in cert.pem -noout -text
```

## 2. Common attacks

### 2.1 Malware types

| Type | Description |
| --- | --- |
| Virus / worm | Virus needs a host file and user action; worm self-propagates across networks |
| Trojan | Disguised as legitimate software; often installs backdoors or RATs |
| Ransomware | Encrypts data and demands payment; modern variants also steal data (double extortion) |
| Rootkit | Hides malicious presence at OS or kernel level |
| Keylogger / infostealer | Captures keystrokes, credentials, browser cookies |
| Fileless malware | Runs in memory using built-in tools (PowerShell, WMI) to avoid disk detection |
| Botnet | Network of infected machines controlled through command and control (C2) |

### 2.2 Social engineering and network attacks

- **Phishing / spear phishing / whaling** – deceptive messages aimed at credentials or malware delivery; targeted at individuals or executives
- **Vishing and smishing** – phishing by voice call or SMS
- **Business Email Compromise (BEC)** – impersonating executives or vendors to redirect payments
- **Man-in-the-middle (MITM)** – attacker intercepts or alters communication (ARP spoofing, rogue Wi-Fi)
- **DoS / DDoS** – overwhelming a service; types include volumetric, protocol (SYN flood) and application layer
- **Password attacks** – brute force, dictionary, credential stuffing, password spraying
- **Privilege escalation** – moving from low to higher privileges (vertical) or to other accounts (horizontal)
- **Lateral movement** – pivoting through the network using stolen credentials and legitimate tools (RDP, SMB, WMI, PsExec)
- **Pass-the-hash** – using a captured NTLM hash to authenticate without knowing the password
- **Kerberoasting** – requesting service tickets for accounts with SPNs and cracking them offline

## 3. Web application security (OWASP Top 10)

The OWASP Top 10 is the standard list of critical web application risks. The list is revised periodically, so check owasp.org for the current edition before your interview. The core risk categories below appear in every edition in some form.

| Risk | What it is | Key prevention |
| --- | --- | --- |
| Broken access control | Users act outside their permissions (IDOR, forced browsing) | Server-side authorization checks, deny by default |
| Cryptographic failures | Weak or missing encryption of sensitive data | TLS everywhere, strong algorithms, proper key management |
| Injection | Untrusted input interpreted as commands (SQL, OS, LDAP) | Parameterised queries, input validation, least-privilege DB accounts |
| Insecure design | Missing security controls at design stage | Threat modelling, secure design patterns |
| Security misconfiguration | Default credentials, verbose errors, open cloud storage | Hardening baselines, automated config checks |
| Vulnerable components | Outdated libraries with known flaws | SCA tools, patching, SBOM |
| Authentication failures | Weak passwords, no MFA, session flaws | MFA, rate limiting, secure session handling |
| Integrity failures | Unsigned updates, insecure deserialization, CI/CD tampering | Signing, verification, pipeline security |
| Logging and monitoring failures | Attacks go undetected | Centralised logging, alerting, retention |
| SSRF | Server tricked into requesting internal resources | Allow-lists, block metadata IPs, network segmentation |

### 3.1 Example payloads you should recognise

```text
SQL injection:   ' OR '1'='1' --
                 admin'--
                 1 UNION SELECT username,password FROM users--
XSS:             <script>alert(document.domain)</script>
                 <img src=x onerror=alert(1)>
Path traversal:  ../../../../etc/passwd
Command inject:  ; cat /etc/passwd
SSRF:            http://169.254.169.254/latest/meta-data/
```

### 3.2 Useful tools

- **Burp Suite** – intercepting proxy for testing web apps (Proxy, Repeater, Intruder)
- **OWASP ZAP** – free alternative scanner and proxy
- **sqlmap** – automated SQL injection testing
- **Nikto, Gobuster / ffuf** – web server scanning and directory brute forcing

## 4. Security frameworks

- **MITRE ATT&CK** – knowledge base of adversary tactics and techniques (Initial Access, Execution, Persistence, Privilege Escalation, Defense Evasion, Credential Access, Discovery, Lateral Movement, Collection, Command and Control, Exfiltration, Impact)
- **Cyber Kill Chain (Lockheed Martin)** – Reconnaissance, Weaponization, Delivery, Exploitation, Installation, Command and Control, Actions on Objectives
- **NIST Cybersecurity Framework** – functions Govern, Identify, Protect, Detect, Respond, Recover (Govern was added in version 2.0)
- **ISO/IEC 27001** – international standard for an Information Security Management System (ISMS), certifiable by audit
- **CIS Controls and CIS Benchmarks** – prioritised practical controls and hardening guides
- **OWASP** – application security resources (Top 10, ASVS, Testing Guide)
- **Diamond Model and Pyramid of Pain** – intelligence analysis concepts; Pyramid of Pain ranks indicators from hashes (easy for attackers to change) to TTPs (hard to change)

## 5. Week 2 interview questions and model answers

### Cryptography

#### Q1. What is the difference between symmetric and asymmetric encryption? When would you use each?

**Answer:** Symmetric uses one shared key for both encryption and decryption; it is fast and suited to bulk data (AES for disk or file encryption). Asymmetric uses a public/private key pair; it is slower but solves key distribution and enables signatures (RSA, ECC).

In practice they are combined: TLS uses asymmetric cryptography to authenticate the server and exchange a key, then symmetric encryption for the session data.

#### Q2. What is the difference between hashing, encryption and encoding?

**Answer:** Encryption is reversible with a key and provides confidentiality. Hashing is one-way and provides integrity. Encoding (Base64) is reversible by anyone and provides no security, only data representation.

#### Q3. How should passwords be stored?

**Answer:** Never in plaintext or with fast hashes like MD5 or plain SHA-256. Use a slow, salted, purpose-built algorithm: Argon2id, bcrypt or scrypt (PBKDF2 where required for compliance). Salting defeats rainbow tables and the work factor slows brute force. Add MFA and breached-password checks on top.

#### Q4. How do digital signatures and certificates work?

**Answer:** The signer hashes the data and encrypts the hash with their private key. Anyone can verify using the public key: decrypt the signature, compare it with a fresh hash of the data. A certificate binds a public key to an identity and is signed by a certificate authority, forming a chain of trust up to a trusted root.

#### Q5. What is a man-in-the-middle attack and how does HTTPS mitigate it?

**Answer:** The attacker positions between client and server to read or modify traffic, for example through ARP spoofing, rogue Wi-Fi or DNS spoofing. HTTPS authenticates the server by certificate and encrypts the channel, so a MITM cannot read or alter data without triggering a certificate warning. HSTS and certificate pinning further reduce downgrade and rogue-certificate risk.

#### Q6. What is Perfect Forward Secrecy?

**Answer:** PFS uses ephemeral key exchange (ECDHE) so each session has a unique key. If the server's long-term private key is later stolen, previously captured traffic still cannot be decrypted.

### Attacks

#### Q7. What is phishing and how would you defend against it?

**Answer:** Phishing is deceiving users into revealing credentials or running malware, usually through email. Defenses are layered:

- Email security: SPF, DKIM, DMARC, sandboxing attachments, URL rewriting
- Phishing-resistant MFA (FIDO2) so stolen passwords are not enough
- User awareness training and an easy "report phishing" button
- Endpoint protection and web filtering
- Fast response process to purge messages and reset credentials

#### Q8. What are SPF, DKIM and DMARC?

**Answer:** SPF lists which servers may send mail for a domain. DKIM adds a cryptographic signature so receivers can verify the message was not altered and came from the domain. DMARC tells receivers what to do if SPF/DKIM checks fail (none, quarantine or reject) and gives reporting.

#### Q9. How does a DDoS attack work and how do you mitigate it?

**Answer:** Many compromised hosts flood a target with traffic or requests to exhaust bandwidth, connection tables or application resources. Mitigation: upstream scrubbing and CDN services, rate limiting, SYN cookies, anycast, auto-scaling, WAF rules for application-layer floods, and an incident runbook with the ISP.

#### Q10. Explain brute force, dictionary, credential stuffing and password spraying.

**Answer:** Brute force tries all combinations; dictionary tries likely words; credential stuffing replays username/password pairs leaked from other breaches; password spraying tries a few common passwords against many accounts to avoid lockouts. Defenses: MFA, lockout or throttling, breached-password screening, monitoring for many failures across accounts (event 4625).

#### Q11. What is pass-the-hash and how can it be prevented?

**Answer:** In NTLM authentication the hash itself is enough to authenticate, so an attacker with admin rights can dump hashes from memory (for example from LSASS) and reuse them to move laterally. Prevention: disable NTLM where possible, enable Credential Guard, LAPS for unique local admin passwords, restrict admin logons to tiered workstations, and monitor for abnormal logons.

#### Q12. What is Kerberoasting?

**Answer:** Any domain user can request service tickets for accounts with Service Principal Names. The tickets are encrypted with the service account's password hash, so the attacker cracks them offline. Defenses: long random passwords or group Managed Service Accounts, AES encryption, and monitoring for unusual spikes in event 4769 with RC4 encryption.

#### Q13. What is ransomware and how would you respond to an outbreak?

**Answer:** Ransomware encrypts files and demands payment, often after stealing data. Response: isolate affected hosts and network segments immediately, preserve evidence (memory and disk if feasible), identify patient zero and the entry vector, disable compromised accounts, check backups for integrity, engage legal and leadership, restore from clean backups, and remediate the root cause. Prevention includes tested offline backups, patching, EDR, MFA and segmentation.

#### Q14. What is lateral movement and how can you detect it?

**Answer:** It is the attacker moving from the initial foothold to other systems, typically with valid credentials and built-in tools. Detection: unusual RDP, SMB or WinRM connections between workstations, logon type 3 or 10 from unexpected hosts, new services created remotely (event 7045), PsExec use, and account logons at odd times. Segmentation and least privilege limit it.

### Web security

#### Q15. How does SQL injection work and how do you prevent it?

**Answer:** The application builds a query by concatenating user input, so an attacker can alter query logic, for example entering ' OR '1'='1 to bypass login or UNION queries to read other tables.

Prevention: parameterised queries or prepared statements (primary control), ORM use, input validation, least-privilege database accounts, suppressing detailed errors and a WAF as an extra layer.

#### Q16. What are the types of XSS and how do you prevent them?

**Answer:** Stored XSS: the payload is saved on the server and served to every visitor. Reflected XSS: the payload is in a request (such as a URL parameter) and echoed back. DOM-based XSS: client-side JavaScript writes untrusted data into the DOM unsafely.

Prevention: context-aware output encoding, input validation, Content Security Policy, HttpOnly and Secure cookie flags, and avoiding dangerous sinks like innerHTML.

#### Q17. What is CSRF and how is it different from XSS?

**Answer:** CSRF tricks a logged-in user's browser into sending an unwanted request to a site that trusts the session cookie (for example, transferring funds). XSS injects script into the trusted site itself. Prevention for CSRF: anti-CSRF tokens, SameSite cookies, and re-authentication for sensitive actions.

#### Q18. What is SSRF?

**Answer:** Server-Side Request Forgery makes the server fetch a URL chosen by the attacker, reaching internal services or cloud metadata endpoints (such as 169.254.169.254) that are not exposed externally. Prevention: allow-list destinations, block internal and link-local ranges, use IMDSv2 in AWS, and segment networks.

#### Q19. What is IDOR / broken access control? Give an example.

**Answer:** Insecure Direct Object Reference occurs when the app exposes an identifier and fails to check ownership: changing /invoice?id=1001 to 1002 shows another customer's invoice. Fix with server-side authorization on every request and avoid relying on hidden or guessable identifiers.

#### Q20. What are secure cookie flags?

**Answer:** HttpOnly stops JavaScript from reading the cookie (limits XSS theft), Secure sends it only over HTTPS, and SameSite restricts cross-site sending (helps against CSRF).

#### Q21. What is the difference between a WAF, IDS and IPS?

**Answer:** A WAF filters HTTP/HTTPS traffic to web applications at Layer 7. An IDS monitors traffic and alerts on suspicious activity but does not block. An IPS sits inline and can block. Each can be signature-based or anomaly-based, and each can produce false positives that need tuning.

### Frameworks

#### Q22. What is MITRE ATT&CK and how is it used in a SOC?

**Answer:** ATT&CK is a public matrix of real-world adversary tactics (the why), techniques (the how) and sub-techniques. SOC teams use it to map detections and measure coverage gaps, structure threat hunts, enrich alerts, and communicate incidents consistently. Example: Kerberoasting maps to Credential Access, technique T1558.003.

#### Q23. Explain the Cyber Kill Chain.

**Answer:** Seven stages: Reconnaissance, Weaponization, Delivery, Exploitation, Installation, Command and Control, Actions on Objectives. Defenders aim to break the chain as early as possible. Its limitation is that it is linear and perimeter-focused, which is why ATT&CK is often used alongside it.

#### Q24. What is the difference between NIST CSF and ISO 27001?

**Answer:** NIST CSF is a voluntary risk-based framework organised around functions (Govern, Identify, Protect, Detect, Respond, Recover) with no certification. ISO 27001 is a certifiable standard specifying requirements for an ISMS, with an audited set of controls. Many organisations use NIST CSF to guide strategy and ISO 27001 for formal assurance.

#### Q25. What is the Pyramid of Pain?

**Answer:** It ranks indicators by how much pain it causes an attacker when you block them: hash values (trivial), IP addresses, domain names, network and host artifacts, tools, and TTPs (hardest). Defenders should aim for detections at the top, since attackers cannot easily change behaviour.

## 6. Week 2 self-check

- [ ] I can explain how TLS combines asymmetric and symmetric cryptography
- [ ] I know how passwords should be stored and why
- [ ] I can describe 10 attack types with detection and prevention for each
- [ ] I can explain SQLi, XSS, CSRF, SSRF and IDOR with an example and a fix
- [ ] I can name the main ATT&CK tactics in order and map two attacks to techniques
- [ ] I can explain the difference between NIST CSF and ISO 27001
- [ ] I completed at least six PortSwigger labs
