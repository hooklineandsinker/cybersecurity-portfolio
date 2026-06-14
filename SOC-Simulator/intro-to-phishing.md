# SOC Simulation Write-Up: Introduction to Phishing
**Platform:** TryHackMe SOC Simulator  
**Date:** June 14, 2026  
**Scenario:** Introduction to Phishing  
**Difficulty:** Beginner  
**Final Score:** 160 points | 1st Place  

---

## Scenario Overview
**Description:** Introduction to Phishing — identify and close 
all True Positive alerts to pass.

**Objectives:**
1. Monitor and analyze real-time alerts
2. Identify and document critical events including suspicious 
   emails and attachments
3. Create detailed case reports documenting the full scope 
   of alerts and malicious activity

---

## SOC Analyst Workflow
1. **Triage** — Identify all true positives in the alert queue
2. **Documentation review** — Consult alert triage & tool docs
3. **Alert queue investigation** — Prioritize by severity
4. **Alert ownership** — Assign alerts; MTTR timer begins
5. **SIEM & VM analysis** — Correlate logs, sandbox attachments
6. **Case reporting** — Classify TP/FP, document findings
7. **Resolution** — All alerts closed, dashboard cleared

---

## Alert Queue Summary

| Alert ID | Severity | Type | Classification | Time to Resolve |
|---|---|---|---|---|
| 8816 | High | Firewall | ✅ True Positive | 10.47 min |
| 8814 | Medium | Phishing | ❌ False Positive | 10.23 min |
| 8817 | Medium | Phishing | ✅ True Positive | 10 min |
| 8815 | Medium | Phishing | ✅ True Positive | 6.83 min |
| 8818 | Medium | Phishing | ✅ True Positive | — |

---

## Investigation & Case Reports

### Alert 8816 — High Severity (Firewall)
**Rule:** Access to Blacklisted External URL Blocked by Firewall  
**Timestamp:** 06/14/2026 19:08:05.160  
**Classification:** True Positive ✅  

**IOCs:**
- Source IP: 10.20.2.17
- Destination IP: 67.199.248.11
- Malicious URL: http://bit.ly/3sHkX3da12340
- Port: 80 (unencrypted HTTP)
- Protocol: TCP
- Firewall rule triggered: Blocked Websites
- Technique: URL shortener obfuscation (bit.ly)

**Analysis:** Internal host 10.20.2.17 attempted to access a 
blacklisted external URL obfuscated via bit.ly shortener. 
Firewall successfully blocked the connection. This alert 
correlates with phishing emails 8815-8818 — the user likely 
clicked a malicious link from one of those emails.

**Remediation:** Identify user on 10.20.2.17. Investigate for 
additional connection attempts. Scan host for malware. Block 
bit.ly/3sHkX3da12340 at perimeter. Re-educate user on phishing 
awareness.

---

### Alert 8814 — Medium Severity (Phishing)
**Rule:** Inbound Email Containing Suspicious External Link  
**Timestamp:** 06/14/2026 19:03:38.160  
**Classification:** False Positive ❌ (analyst initially 
classified as TP — see Lessons Learned)

**IOCs:**
- Sender: onboarding@hrconnex.thm
- Recipient: j.garcia@thetrydaily.thm
- Subject: "Action Required: Finalize Your Onboarding Profile"
- URL: hrconnex.thm/onboarding/154006S4060/j.garcia
- Technique: HR impersonation, urgency language

**Analysis:** Email appeared to be HR onboarding phishing due 
to external domain and urgency language. Correct classification 
was False Positive — hrconnex.thm is a legitimate third-party 
HR onboarding platform used by the organization.

---

### Alert 8815 — Medium Severity (Phishing)
**Rule:** Inbound Email Containing Suspicious External Link  
**Timestamp:** 06/14/2026 19:06:51.160  
**Classification:** True Positive ✅  

**IOCs:**
- Sender: urgents@amazon.biz
- Recipient: h.harris@thetrydaily.thm
- Subject: "Your Amazon Package Couldn't Be Delivered – 
  Action Required"
- Malicious URL: http://bit.ly/3sHkX3da12340
- Technique: Amazon delivery impersonation, 48-hour deadline 
  urgency, same malicious URL as firewall alert 8816

**Analysis:** Phishing email impersonating Amazon delivery 
notification. Sender domain amazon.biz is not a legitimate 
Amazon domain. Contains the same bit.ly URL that triggered 
firewall alert 8816 — confirming this email as the likely 
source of that connection attempt.

**Remediation:** Block sender domain amazon.biz. Notify 
h.harris. Correlate with host 10.20.2.17 to confirm user 
identity.

---

### Alert 8817 — Medium Severity (Phishing)
**Rule:** Inbound Email Containing Suspicious External Link  
**Timestamp:** 06/14/2026 19:09:09.160  
**Classification:** True Positive ✅  

**IOCs:**
- Sender: no-reply@m1crosoftsupport.co
- Recipient: c.allen@thetrydaily.thm
- Subject: "Unusual Sign-In Activity on Your Microsoft Account"
- Malicious URL: https://m1crosoftsupport.co/login
- Claimed source IP in email: 102.89.222.143 (Lagos, Nigeria)
- Technique: Microsoft impersonation, typosquatting 
  (m1crosoftsupport.co — "i" replaced with "1"), 
  fake security alert urgency

**Analysis:** Phishing email impersonating Microsoft security 
alert. Typosquatted domain designed to deceive recipients 
visually. Fake login page used to harvest Microsoft credentials. 
Urgency created by claiming unauthorized access from Nigeria.

**Remediation:** Block m1crosoftsupport.co. Notify c.allen 
immediately. Check proxy logs for access to 
m1crosoftsupport.co/login. Force Microsoft password reset 
for c.allen as precaution.

---

### Alert 8818 — Medium Severity (Phishing)
**Rule:** Inbound Email Containing Suspicious External Link  
**Timestamp:** 06/14/2026 19:11  
**Classification:** True Positive ✅  

*(Similar HR onboarding phishing pattern — additional IOCs 
to be added from case report)*

---

## Final Results

| Metric | Result |
|---|---|
| Score | 160 points |
| Leaderboard position | 1st place |
| True Positive identification rate | 100% |
| False Positive identification rate | N/A |
| Closed alerts | 4 |
| Mean time to resolve | 9 minutes |
| Mean dwell time | 28 minutes |

---

## Attack Patterns Identified
1. **HR onboarding impersonation** — external domain mimicking 
   internal HR communications
2. **Delivery notification phishing** — Amazon impersonation 
   with fake package delivery urgency
3. **Microsoft account security alert** — typosquatted domain 
   credential harvesting
4. **URL shortener obfuscation** — bit.ly used to hide 
   malicious destination from email filters
5. **Multi-vector campaign** — phishing emails correlated 
   with firewall alert, suggesting at least one user clicked

---

## Tools Used
- TryHackMe SOC Simulator
- Built-in SIEM
- Analyst VM
- Phishing Playbook
- Case Report System

---

## Lessons Learned

### Alert 8814 — Misclassification
Classified as True Positive but correct answer was False 
Positive. The HR onboarding email from onboarding@hrconnex.thm 
appeared suspicious but was a legitimate third-party service.

**Key takeaway:** Always verify with internal stakeholders 
before classifying ambiguous alerts. External sender domains 
used for legitimate business services are common. Apply the 
5 Ws consistently — Who sent it, What does it contain, When 
was it sent, Where does the link go, Why would this be sent 
to this recipient. Context beyond technical IOCs is essential 
to avoid false positive misclassification.

### Report Writing Feedback (AI Analysis)
Platform feedback noted that reports should more consistently 
address the potential impact and consequences of threats, 
and ensure all 5 Ws are covered in each case report for 
completeness.
