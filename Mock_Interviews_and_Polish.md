# Week 4: Mock Interviews and Polish

> Behavioral, scenario and rapid-fire questions, plus final checklist

[Home](README.md) | [Week 1](Week1_Fundamentals.md) | [Week 2](Week2_Core_Security_Concepts.md) | [Week 3](Week3_Role_Specific_Depth.md) | [Week 4](Week4_Mock_Interviews_and_Polish.md)

---

## How to use this document

Week 4 turns your knowledge into interview performance. The goal is to speak clearly, structure answers and handle scenarios calmly. Do at least three timed mock interviews (with a friend, a recording of yourself, or Claude) and rehearse your resume stories until they feel natural.

| Day | Focus | Task |
| --- | --- | --- |
| Day 1 | Resume and project stories | Prepare 3 STAR stories and a 60-second "tell me about yourself" |
| Day 2 | Behavioral questions | Practise Section 2 questions aloud with a timer |
| Day 3 | Scenario and troubleshooting questions | Work through Section 3 scenarios using a fixed structure |
| Day 4 | Rapid-fire technical revision | Answer Section 4 in under 30 seconds each |
| Day 5 | Full mock interview 1 | Mix technical, scenario and behavioral for 45 to 60 minutes |
| Day 6 | Full mock interview 2 and weak spots | Review recordings; fix filler words and gaps |
| Day 7 | Logistics and rest | Use the final checklist in Section 7 |

## 1. Resume and introduction

### 1.1 "Tell me about yourself" (60 to 90 seconds)

Use a Present, Past, Future structure: who you are now and your focus, the experience and skills that got you here, and why this role is your next step.

```text
"I'm a [background, e.g. recent graduate / IT support engineer] focused on [SOC / pentesting / cloud security].
Over the past year I built a home lab with Splunk and Windows Active Directory, completed the
TryHackMe SOC Level 1 path, and investigated simulated phishing and brute-force incidents.
I enjoy [the investigative side of security / finding vulnerabilities]. I'm now looking for a role
where I can apply this hands-on experience on a real team, and your focus on [something specific
about the company] is exactly what I want to learn from."
```

Replace the bracketed parts with your own facts. Never claim experience you do not have; interviewers probe every line of the resume.

### 1.2 Project story template

- **Goal** – what problem were you trying to solve or learn?
- **Setup** – what tools and architecture did you use?
- **What you did** – two or three specific actions in your words
- **Result** – what you found or detected, ideally with a number
- **Learning** – what you would improve next time

## 2. Behavioral questions and the STAR method

STAR: Situation (context), Task (your responsibility), Action (what YOU did, step by step), Result (outcome, ideally measurable, and what you learned). Keep answers to about two minutes. Use "I" for your own actions, not "we".

#### Q1. Tell me about a time you identified or fixed a security issue.

**Answer:** Sample structure: "In my home lab / at my previous job (Situation), I noticed RDP was exposed with a weak password policy (Task: reduce the risk). I checked logs and found hundreds of failed logons (Action), then restricted RDP behind a VPN, enforced a strong password policy and set up an alert for repeated 4625 events. Afterwards failed logons from the internet dropped to zero and I documented the change as a checklist (Result)."

Tip: choose a real example, even a lab one, and be specific about your own contribution.

#### Q2. Tell me about a time you made a mistake. What did you do?

**Answer:** Choose a genuine, modest mistake, explain how you noticed it, how you fixed it and what process change you made. Example: "I closed an alert as a false positive without checking the user's mailbox rules. When a colleague found a forwarding rule later, I reopened it, reported my miss, and created a checklist to always check for forwarding rules on suspicious-login alerts."

Interviewers want honesty and learning, not perfection.

#### Q3. How do you explain a security risk to a non-technical manager?

**Answer:** I avoid jargon, describe the business impact first (what could be lost, how likely, how much it would cost), use an analogy if helpful, and give a clear recommendation with options and effort required. For example: "An unpatched server is like leaving a door unlocked; attackers actively scan for it. Patching takes two hours; a breach could halt orders for days."

#### Q4. How do you stay up to date with cybersecurity?

**Answer:** Name real sources and habits: The Hacker News, BleepingComputer, Krebs on Security, CISA advisories and the CISA KEV catalogue, MITRE ATT&CK updates, vendor threat reports (Microsoft, CrowdStrike, Mandiant), podcasts, X/LinkedIn researchers, plus regular labs on TryHackMe, HTB or LetsDefend. Mention a recent incident you read about and what you learned from it.

#### Q5. How do you handle pressure during an incident?

**Answer:** I follow the process: stay calm, confirm facts, prioritise containment, communicate clearly and often, and write things down as I go. I ask for help early and avoid making irreversible changes without evidence. Playbooks and checklists reduce stress because they remove guesswork.

#### Q6. Tell me about a time you disagreed with a teammate or manager.

**Answer:** Use a short example in which you listened, presented evidence (logs, data, risk) rather than opinion, and reached a decision or compromise, and the relationship stayed positive. End with the outcome and what you learned about communication.

#### Q7. Why do you want to work in cybersecurity, and why this company?

**Answer:** Give a genuine reason (for example, enjoying investigation and problem solving, wanting to protect people and systems), connect it to concrete things you have done, and refer to something specific about the company: its industry, its products, its security blog or a recent public initiative. Research this beforehand.

#### Q8. What are your strengths and weaknesses?

**Answer:** Strength: pick one relevant to the role with evidence (for example, methodical investigation shown by detailed lab write-ups). Weakness: a real one that you are actively improving (for example, being less confident with scripting, so you are completing a weekly Python automation exercise), not a disguised strength.

## 3. Scenario and troubleshooting questions

For scenarios, think out loud with a consistent structure: 1) clarify and verify, 2) assess scope and severity, 3) contain, 4) investigate root cause, 5) eradicate and recover, 6) communicate and document, 7) prevent recurrence. Interviewers care more about your reasoning than a single perfect answer.

#### Q9. An employee reports clicking a link in a suspicious email and entering their password. What do you do?

**Answer:** I treat it as a credential compromise. Immediately reset the password and revoke active sessions and tokens, check MFA registrations and mailbox rules (forwarding, deletion rules), review sign-in logs for logins from unusual IPs, and check whether the link or attachment delivered malware to the endpoint (EDR scan, isolate if needed).

Then search the mail system for the same message across the organisation and purge it, block the sender and URL, and look for other users who clicked. Finally, document, give the employee supportive feedback and consider updating awareness training or mail filtering.

#### Q10. You discover a server communicating with an unknown external IP every 60 seconds. What does it suggest and what do you do?

**Answer:** Regular intervals suggest beaconing to a C2 server. I would check the IP and domain reputation, identify the process responsible through EDR or netstat/ss, and review when the behaviour began and what changed around that time. I would isolate the host if malicious, capture memory and relevant logs, search the environment for the same indicators, examine credentials used from the host and then remediate (reimage, reset credentials, block IOCs).

#### Q11. A critical vulnerability (for example, a remote code execution flaw in a widely used product) is announced. What do you do?

**Answer:** Identify exposure: use asset inventory and vulnerability scanners to find affected systems, prioritising internet-facing ones. Check whether exploitation is happening in the wild (CISA KEV, vendor advisories). Apply patches or mitigations (WAF rules, disabling a feature, network restrictions) in priority order, hunt for signs of prior exploitation using logs and published IOCs, and communicate status and timelines to stakeholders.

#### Q12. Ransomware has encrypted a file server. What are your first steps?

**Answer:** Contain first: disconnect the affected server and other suspected hosts from the network (do not power off if memory evidence is needed), disable compromised accounts and block known C2. Activate the incident response plan and notify leadership. Determine scope, entry point and whether data was stolen. Confirm backups are clean and offline before restoring. Rebuild affected systems, reset credentials broadly if domain compromise is suspected, then conduct a post-incident review. Decisions about paying a ransom belong to leadership with legal and law-enforcement advice.

#### Q13. You see event ID 1102 (audit log cleared) on a domain controller. What does it mean and what do you do?

**Answer:** Clearing the security log is a strong sign of an attacker covering their tracks, or rarely an admin action. I would identify who cleared it and from where, check the change tickets, review other logs (forwarded SIEM copies, EDR telemetry, network logs) for the activity the attacker may have hidden, and treat the DC as potentially compromised: escalate as a high-severity incident, review privileged account activity and consider credential resets, including krbtgt, if domain compromise is confirmed.

#### Q14. A developer accidentally committed an AWS access key to a public GitHub repository. What do you do?

**Answer:** Treat the key as compromised. Immediately deactivate and rotate it, then review CloudTrail for any use of the key, especially from unknown IPs, and check for new resources, IAM users or unusual spending. Remove the secret from the repo history (and rotate regardless, since forks and caches may hold it). Prevent recurrence with secret scanning (pre-commit hooks, GitHub secret scanning), short-lived roles instead of static keys, and developer training.

#### Q15. How would you secure a new company with no security program, starting from zero?

**Answer:** Start with visibility and the basics: asset and account inventory, MFA on all accounts, patching, endpoint protection, backups tested offline, and centralised logging. Then address least privilege, secure configuration baselines, email security and basic awareness training, and write an incident response plan. Use a framework such as CIS Controls or NIST CSF to prioritise based on risk and revisit periodically.

#### Q16. Users report the company website is very slow. How do you determine whether it is a DDoS attack?

**Answer:** Check traffic and resource metrics: unusual spikes in requests per second, bandwidth or connections, concentration from certain IPs, regions or user agents, and the targeted URLs. Compare with normal baselines and marketing events. If it is an attack, enable rate limiting and WAF rules, engage the CDN or DDoS mitigation provider, and contact the ISP. If it is not, investigate application and database performance, a recent deployment or infrastructure problems.

## 4. Rapid-fire technical revision

Try to answer each in under 30 seconds. Short, accurate answers show real understanding.

| Question | Short answer |
| --- | --- |
| What port does RDP use? | TCP 3389 |
| What is the difference between IDS and IPS? | IDS detects and alerts; IPS sits inline and can block |
| What is a zero-day? | A vulnerability exploited before a patch exists |
| What does DMARC do? | Tells receivers how to handle mail failing SPF/DKIM and provides reporting |
| What is salting? | Adding a random per-user value before hashing so equal passwords give different hashes |
| What is a honeypot? | A decoy system used to detect and study attackers |
| What is the principle of least privilege? | Grant only the minimum access needed to do the job |
| What is a CVE and a CVSS score? | CVE is an identifier for a known vulnerability; CVSS rates its severity from 0 to 10 |
| What is tailgating? | Following an authorised person through a secured door without authorisation |
| What is a proxy vs a reverse proxy? | A forward proxy sits in front of clients; a reverse proxy sits in front of servers |
| What is a VPN? | An encrypted tunnel between a device and a network over an untrusted network |
| What is data at rest vs in transit? | Stored data (disk, database) vs data moving across a network |
| What is a sandbox? | An isolated environment for safely running suspicious files |
| What is threat hunting? | Proactively searching for attackers using hypotheses, not waiting for alerts |
| What is XDR? | Extended detection and response, correlating telemetry across endpoints, email, network and cloud |
| What is Zero Trust? | Never trust by default; verify every user, device and request continuously and enforce least privilege |
| What is an air-gapped system? | A system physically isolated from other networks |
| What is a rainbow table? | A precomputed table of hashes used to reverse unsalted hashes quickly |
| What is DLP? | Data loss prevention: tools and policies that detect and block sensitive data leaving the organisation |
| What is a CASB? | Cloud access security broker: enforces security policy between users and cloud services |

## 5. Questions to ask the interviewer

- What does a typical day or week look like for someone in this role?
- What tools make up the security stack (SIEM, EDR, ticketing, cloud)?
- How is the security team structured and who would I work with most closely?
- What are the biggest security challenges the organisation is facing right now?
- How does the team handle learning, certifications and career progression?
- What would success look like in the first 90 days?
- Is there on-call or shift work, and how is it managed?

## 6. Mock interview script (use with a friend or Claude)

| Round | Time | Content |
| --- | --- | --- |
| Introduction | 5 min | Tell me about yourself; why this role |
| Fundamentals | 10 min | 5 questions from Weeks 1 and 2 (networking, crypto, attacks) |
| Role-specific | 15 min | 3 to 4 questions from your Week 3 track |
| Scenario | 10 min | One incident scenario, talked through end to end |
| Behavioral | 10 min | 2 STAR questions |
| Your questions | 5 min | Ask two prepared questions |

After each mock, score yourself on: accuracy, structure, clarity, confidence, and use of specific examples. Write down three things to improve before the next one.

## 7. Final preparation checklist

### A week before

- [ ] Resume updated; every tool and project listed is something you can discuss in depth
- [ ] GitHub, blog or write-ups tidied and linked on the resume
- [ ] Company research done: products, recent news, security incidents, technology stack from job posts
- [ ] Read the job description line by line and match each requirement to an example from your experience

### The day before

- [ ] Re-read your cheat sheets: ports, event IDs, OWASP Top 10, IR phases, ATT&CK tactics
- [ ] Prepare your questions for the interviewer
- [ ] Test your video call setup, internet, microphone and lighting for online interviews
- [ ] Sleep well; last-minute cramming has little value

### During the interview

- [ ] Listen to the full question and take a moment to think before answering
- [ ] Structure answers: short definition, how it works, example, mitigation
- [ ] Think aloud on scenarios and state your assumptions
- [ ] If you do not know, say so, explain how you would find out and give the closest related knowledge
- [ ] Be honest about lab versus production experience; show eagerness to learn
- [ ] Send a short thank-you message afterwards

### Common mistakes to avoid

- [ ] Memorising definitions without being able to give an example
- [ ] Claiming expertise in tools you have only read about
- [ ] Skipping the "verify first" step in scenario answers and jumping to conclusions
- [ ] Forgetting containment and communication in incident answers
- [ ] Rambling for more than two to three minutes on one answer

## 8. Week 4 self-check

- [ ] I have a polished 60 to 90 second introduction
- [ ] I have three STAR stories ready and timed
- [ ] I can handle a scenario question using a consistent structure
- [ ] I completed at least two full mock interviews
- [ ] I have five questions prepared for the interviewer
- [ ] I have reviewed all four weeks of notes at least once
