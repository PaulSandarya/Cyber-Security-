# Week 3: Role-Specific Depth and Hands-On Practice

> SOC, red team, cloud and GRC tracks, with interview Q&A

[Home](README.md) | [Week 1](Week1_Fundamentals.md) | [Week 2](Week2_Core_Security_Concepts.md) | [Week 3](Week3_Role_Specific_Depth.md) | [Week 4](Week4_Mock_Interviews_and_Polish.md)

---

## How to use this document

Week 3 is role-specific. Focus on the track that matches the job you are applying for. If you are undecided, cover the SOC track fully (most entry-level roles) and skim the others. Hands-on practice matters most this week: interviewers can tell quickly who has actually used the tools.

| Day | Track | Hands-on task |
| --- | --- | --- |
| Day 1 | SOC / Blue team: incident response and SIEM | Install Splunk Free or Wazuh in a lab; ingest Windows and Linux logs |
| Day 2 | SOC: alert triage, EDR, phishing analysis | LetsDefend or TryHackMe SOC Level 1 alert investigations |
| Day 3 | Red team / pentest: methodology and tools | Nmap and Gobuster scans against a TryHackMe or HTB machine |
| Day 4 | Pentest: privilege escalation and reporting | Complete one easy box and write a short report |
| Day 5 | Cloud security | Review AWS IAM policies; build a free-tier lab and enable CloudTrail |
| Day 6 | GRC: risk, policy, compliance | Create a simple risk register for a fictional company |
| Day 7 | Revision and Q&A out loud | Answer all questions for your target track |

## Track A: SOC analyst / Blue team

### A.1 Incident response lifecycle (NIST SP 800-61)

| Phase | Goal | Example actions |
| --- | --- | --- |
| 1. Preparation | Be ready before incidents happen | IR plan, playbooks, logging, contact lists, tooling, training |
| 2. Detection and analysis | Confirm and scope the incident | Triage alerts, correlate logs, determine severity and affected assets |
| 3. Containment, eradication, recovery | Stop spread, remove threat, restore service | Isolate hosts, disable accounts, remove persistence, restore from clean backups, monitor |
| 4. Post-incident activity | Learn and improve | Lessons-learned review, update detections and playbooks, report |

Many interviewers use the older SANS six-step version: Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned. Know both names and be consistent in how you present them.

### A.2 SIEM and log analysis

- **SIEM** – collects, normalises, correlates and alerts on logs from across the environment (Splunk, Microsoft Sentinel, Elastic, QRadar, Wazuh)
- **Use cases** – brute force detection, impossible travel, new admin account, malware beaconing, data exfiltration, log clearing
- **Log sources** – firewall, proxy, DNS, EDR, domain controllers, VPN, cloud audit logs (CloudTrail, Azure Activity), email gateway
- **False positive** – alert with no real threat; false negative :: real threat that was not detected. Tuning reduces the first without causing the second
- **SOAR** – automation of repetitive response steps (enrich IP, block domain, disable user)

### A.3 Example Splunk searches

```text
# Failed Windows logons by source (possible brute force)
index=wineventlog EventCode=4625
| stats count by src_ip, user
| where count > 10
| sort - count

# Successful logon after many failures
index=wineventlog (EventCode=4625 OR EventCode=4624)
| stats count(eval(EventCode=4625)) as fails, count(eval(EventCode=4624)) as success by user, src_ip
| where fails > 5 AND success > 0

# New process with encoded PowerShell
index=wineventlog EventCode=4688 CommandLine="*-enc*"
| table _time, host, user, ParentProcessName, CommandLine

# Possible beaconing: rare, regular outbound connections
index=proxy
| stats count, dc(url) as urls by src_ip, dest_ip
| where count > 200 AND urls < 3
```

### A.4 Alert triage checklist

- **Understand the alert** – what rule fired, what is the data source, what is the claimed behaviour
- **Scope** – which user, host, IP and time window; any related alerts
- **Enrich** – reputation (VirusTotal, AbuseIPDB), asset criticality, user role, geolocation
- **Validate** – is it expected activity (change ticket, admin task, scanner)?
- **Decide** – false positive, true positive (escalate), or needs more data
- **Document** – findings, evidence, actions and reasoning in the ticket

### A.5 Phishing email analysis steps

- **Check headers** – From, Return-Path, Received chain, SPF/DKIM/DMARC results
- **Inspect links and attachments** – hover URLs, detonate in a sandbox, hash attachments and check reputation
- **Search the mail platform** – who else received it and who clicked
- **Respond** – purge messages, block sender and URLs, reset credentials of anyone who entered them, hunt for follow-on activity

### A.6 SOC Q&A

#### Q1. Walk me through the incident response lifecycle.

**Answer:** Preparation (plans, tooling, training), Detection and Analysis (identify and scope), Containment, Eradication and Recovery (isolate, remove the threat, restore, verify) and Post-Incident Activity (lessons learned and improvements). I would add that these phases overlap in practice; for example, analysis continues during containment as new indicators appear.

#### Q2. You get an alert for a successful login from another country. What do you do?

**Answer:** First I verify rather than assume. I check the user, the source IP and geolocation, whether MFA was used, device and user-agent, and whether the user travels or uses a VPN.

Then I look for post-login activity: mailbox rule creation, forwarding rules, unusual downloads, new MFA devices, access to sensitive apps. I compare with the user's normal baseline and check for other logins from the same IP.

If suspicious: contact the user through a separate channel, revoke sessions and tokens, reset the password and MFA, and escalate as an incident. If benign, I document it and tune the rule if appropriate.

#### Q3. A user's laptop is beaconing to a known malicious IP. What are your next steps?

- Confirm the alert in proxy, firewall and EDR data; identify the process making the connection
- Contain: isolate the host with EDR network containment (keep it powered on to preserve memory)
- Scope: search the environment for the same IP, domain, hash and process; check for lateral movement and credential use
- Investigate: how it arrived (email, download, exploit), persistence mechanisms, data accessed
- Eradicate and recover: reimage the host, reset credentials, block IOCs, restore service
- Document and improve detections

#### Q4. How do you prioritise 50 alerts arriving at once?

**Answer:** I prioritise by severity, asset criticality and confidence: alerts on domain controllers, privileged accounts or internet-facing systems come first, along with alerts that correlate with others. Known noisy rules are handled last or batched. I escalate anything showing active compromise immediately and keep a record of what I deferred and why. Over time I raise recurring noise for tuning.

#### Q5. What is the difference between a false positive and a false negative? Which is worse?

**Answer:** A false positive is a benign event flagged as malicious; a false negative is a real attack missed. False negatives are generally more dangerous because the threat goes unnoticed, but too many false positives cause alert fatigue and lead analysts to miss real threats, so both must be managed.

#### Q6. What is EDR and how does it differ from antivirus?

**Answer:** Traditional antivirus mainly uses signatures to block known malware. EDR continuously records endpoint telemetry (process creation, network connections, file and registry changes), detects behavioural patterns, and gives responders tools to investigate and contain (isolate host, kill process, collect files). EDR is oriented to detection and response, not just prevention.

#### Q7. What are IOCs and IOAs?

**Answer:** Indicators of Compromise are artefacts showing a compromise has occurred (hashes, IPs, domains, registry keys). Indicators of Attack focus on attacker behaviour and intent (for example, a Word process spawning PowerShell). IOAs are more durable because attackers change IOCs easily.

#### Q8. How would you investigate a suspected brute force attack?

**Answer:** Query authentication logs for many failures (4625 on Windows, auth.log on Linux) grouped by source IP and target account. Check whether a success (4624) followed the failures, the logon type and source. If successful, treat the account as compromised: reset credentials, review activity after login and look for lateral movement. Block the source and consider rate limiting, lockout policy and MFA.

#### Q9. What would you look for to detect data exfiltration?

**Answer:** Large or unusual outbound transfers, uploads to personal cloud storage, connections to rare external hosts, DNS tunnelling (long or high-volume queries), use of archive tools before transfer, off-hours activity, and DLP alerts. Baselining per user and host makes anomalies easier to spot.

## Track B: Red team / Penetration tester

### B.1 Penetration testing methodology

- **Pre-engagement** – scope, rules of engagement, authorisation in writing, contacts and timelines
- **Reconnaissance** – passive (OSINT, DNS, certificates) and active (scanning)
- **Scanning and enumeration** – ports, services, versions, users, shares, web content
- **Vulnerability analysis and exploitation** – validate and exploit weaknesses safely
- **Post-exploitation** – privilege escalation, persistence (if in scope), lateral movement, data access proof
- **Reporting** – executive summary, technical findings with evidence, risk rating, remediation advice, retest

### B.2 Core commands

```text
# Nmap
nmap -sC -sV -oA scan 10.10.10.5          # default scripts + version detection
nmap -p- --min-rate 1000 10.10.10.5       # all TCP ports
nmap -sU --top-ports 50 10.10.10.5        # common UDP ports

# Web enumeration
gobuster dir -u http://target -w /usr/share/wordlists/dirb/common.txt
nikto -h http://target

# SMB / services
smbclient -L //10.10.10.5 -N
enum4linux -a 10.10.10.5

# Password attacks (authorised labs only)
hydra -l admin -P rockyou.txt ssh://10.10.10.5
hashcat -m 1000 hashes.txt rockyou.txt
```

### B.3 Privilege escalation checklist

- **Linux** – sudo -l, SUID binaries, writable cron jobs, kernel version, weak file permissions, credentials in files and history; tools LinPEAS and GTFOBins
- **Windows** – whoami /priv, unquoted service paths, weak service permissions, AlwaysInstallElevated, stored credentials, token impersonation; tools WinPEAS and PowerUp
- **Active Directory** – Kerberoasting, AS-REP roasting, ACL abuse, delegation misconfigurations; tools BloodHound and Impacket

### B.4 Pentest Q&A

#### Q10. Walk me through how you would approach a penetration test of a web application.

**Answer:** I start by confirming scope and rules of engagement. Then I map the application (pages, parameters, APIs, roles) with a proxy such as Burp Suite, enumerate technologies and hidden content, and test systematically against the OWASP Top 10: authentication and session handling, access control (IDOR, privilege escalation across roles), injection, XSS, CSRF, file upload, business logic and misconfigurations.

I validate each finding with a safe proof of concept, record evidence and impact, and write a report with prioritised remediation.

#### Q11. What is the difference between a vulnerability assessment and a penetration test?

**Answer:** A vulnerability assessment identifies and ranks weaknesses, typically with automated scanners, and is broad. A penetration test goes further by attempting to exploit them to show real impact and attack paths, and is more manual and targeted.

#### Q12. Explain black box, grey box and white box testing.

**Answer:** Black box: no prior knowledge of the target. Grey box: partial knowledge, such as a normal user account. White box: full knowledge, including source code and architecture. Grey and white box usually find more issues in less time.

#### Q13. What does an Nmap -sS scan do, and how is it different from -sT?

**Answer:** -sS is a SYN (half-open) scan: it sends SYN, and on SYN-ACK it sends RST without completing the handshake. It is faster and was historically stealthier, but needs raw-socket (root) privileges. -sT completes the full TCP connect and works without privileges but is more easily logged.

#### Q14. You have a low-privilege shell on a Linux box. How do you escalate?

**Answer:** I enumerate: id, sudo -l, kernel and OS versions, SUID/SGID files, cron jobs and writable scripts, running services and local ports, environment variables, and readable config or history files for credentials. I check GTFOBins for any usable sudo/SUID binaries and run LinPEAS for coverage. I choose the safest technique first, avoiding kernel exploits on production-like systems unless explicitly permitted.

#### Q15. How do you write a good penetration test report?

**Answer:** An executive summary in business terms, scope and methodology, findings ordered by risk (each with description, evidence/screenshots, impact, likelihood, CVSS or similar rating and clear remediation), and a conclusion with retest recommendations. Findings must be reproducible and written so both technical staff and management can act on them.

#### Q16. What is the difference between red team, blue team and purple team?

**Answer:** Red team simulates real adversaries to test detection and response. Blue team defends and responds. Purple team is a collaborative exercise where red and blue share techniques and results in real time to improve detections quickly.

## Track C: Cloud security

### C.1 Key concepts

- **Shared responsibility model** – the provider secures the cloud itself (hardware, hypervisor, facilities); the customer secures what they put in it (identities, data, configuration, workloads). The split shifts across IaaS, PaaS and SaaS
- **IAM** – users, roles, groups and policies; apply least privilege, prefer roles over long-lived keys, enforce MFA, review permissions regularly
- **Network controls** – VPC/VNet segmentation, security groups and NACLs, private endpoints, WAF
- **Data protection** – encryption at rest (KMS) and in transit, bucket policies, block public access, backups
- **Logging** – CloudTrail, Config, GuardDuty (AWS); Activity Log, Defender for Cloud (Azure); Cloud Audit Logs, SCC (GCP)
- **Common misconfigurations** – public storage buckets, overly permissive IAM (wildcards), exposed management ports, unrotated access keys, disabled logging, secrets in code
- **Infrastructure as code (IaC)** – scan Terraform and CloudFormation templates before deployment (Checkov, tfsec)
- **CSPM / CWPP / CNAPP** – posture management, workload protection and combined cloud-native security platforms

### C.2 Cloud Q&A

#### Q17. Explain the shared responsibility model.

**Answer:** The cloud provider is responsible for security of the cloud: physical data centres, hardware and the virtualisation layer. The customer is responsible for security in the cloud: IAM, data classification and encryption, network configuration, OS and application patching on IaaS, and logging. With SaaS the customer retains responsibility mainly for identities, data and access settings.

#### Q18. An S3 bucket was found publicly accessible. What do you do?

**Answer:** Immediately block public access at bucket and account level, then review the bucket policy and ACLs. Use CloudTrail and S3 access logs to determine whether data was accessed and by whom. Assess what data was exposed, follow breach notification obligations if needed, rotate any exposed credentials, and prevent recurrence with Block Public Access enforced by service control policies, AWS Config rules and IaC scanning.

#### Q19. How do you secure IAM in AWS?

**Answer:** Enforce MFA, remove root usage, apply least privilege with scoped policies (no wildcard actions or resources), use roles with temporary credentials rather than long-lived access keys, use permission boundaries and SCPs in AWS Organizations, review access with IAM Access Analyzer, and rotate or remove unused credentials.

#### Q20. What is the difference between a security group and a network ACL?

**Answer:** Security groups are stateful, attached to instances or network interfaces, and allow-only. Network ACLs are stateless, applied at the subnet level, and support both allow and deny rules, so return traffic must be explicitly permitted.

#### Q21. How can an SSRF vulnerability lead to a cloud compromise?

**Answer:** The attacker makes the application request the instance metadata service (169.254.169.254) and retrieves temporary IAM role credentials, then uses them to access cloud resources. Mitigations: enforce IMDSv2, restrict the instance role's permissions, validate outbound requests and monitor for use of credentials from unexpected IPs.

## Track D: GRC (Governance, Risk and Compliance)

### D.1 Key concepts

- **Risk assessment** – identify assets, threats and vulnerabilities, estimate likelihood and impact, rate risk, and decide treatment
- **Risk treatment options** – mitigate, transfer (insurance), accept, avoid
- **Qualitative vs quantitative** – ratings such as High/Medium/Low vs monetary values. Quantitative formulas: SLE = asset value x exposure factor; ALE = SLE x ARO (annual rate of occurrence)
- **Policies, standards, procedures, guidelines** – policy states intent, standards set mandatory requirements, procedures give steps, guidelines are recommendations
- **Compliance frameworks** – ISO 27001, SOC 2, PCI DSS (card data), HIPAA (US health data), GDPR (EU personal data), India's Digital Personal Data Protection Act 2023
- **Audit types** – internal, external, first-party, third-party; evidence-based assessment against criteria
- **Business continuity and disaster recovery** – BIA, RTO (recovery time objective), RPO (recovery point objective)
- **Third-party risk** – vendor questionnaires, contractual security clauses, ongoing monitoring

### D.2 GRC Q&A

#### Q22. How would you perform a risk assessment?

**Answer:** Define scope and identify critical assets, then identify threats and vulnerabilities for each. Estimate likelihood and impact using an agreed scale, calculate risk ratings, and record them in a risk register with owners. Recommend treatments (mitigate, transfer, accept, avoid), get management decisions, track remediation, and reassess periodically or when the environment changes.

#### Q23. What is the difference between a policy, a standard and a procedure?

**Answer:** A policy is a high-level management statement of intent and rules. A standard defines specific mandatory requirements that support the policy (for example, minimum password length). A procedure gives step-by-step instructions for performing a task. Guidelines are optional recommendations.

#### Q24. What is the difference between RTO and RPO?

**Answer:** RTO is the maximum acceptable time to restore a service after a disruption. RPO is the maximum acceptable amount of data loss measured in time, which determines how often backups or replication must run.

#### Q25. What is the difference between SOC 2 and ISO 27001?

**Answer:** ISO 27001 certifies that an organisation operates an ISMS meeting the standard and is recognised globally. SOC 2 is an attestation report by a CPA firm on controls relevant to the Trust Services Criteria (security, availability, processing integrity, confidentiality, privacy) and is common for US SaaS vendors. Type 1 covers design of controls at a point in time; Type 2 covers operating effectiveness over a period.

#### Q26. What would you do if a business owner refuses to remediate a high risk?

**Answer:** I would make sure they understand the business impact in plain terms, discuss feasible alternatives or compensating controls, and escalate to the risk committee or senior management if needed. If the owner still accepts the risk, it should be formally documented with approval, an expiry or review date and monitoring, since risk acceptance must be a conscious decision by someone with authority.

#### Q27. What is GDPR and what are its key principles?

**Answer:** The EU General Data Protection Regulation governs processing of personal data of people in the EU. Key principles include lawfulness, fairness and transparency; purpose limitation; data minimisation; accuracy; storage limitation; integrity and confidentiality; and accountability. Notable obligations include breach notification to the regulator within 72 hours and respecting data subject rights (access, erasure, portability).

## Week 3 self-check

- [ ] I can describe the IR lifecycle and apply it to two realistic scenarios
- [ ] I have run at least one SIEM search of my own and can explain it
- [ ] I can explain what EDR, SOAR, IOC and IOA mean
- [ ] I have completed at least one lab machine end to end (for pentest roles)
- [ ] I can explain the shared responsibility model and five common cloud misconfigurations
- [ ] I can explain risk treatment options and the difference between RTO and RPO (for GRC roles)
