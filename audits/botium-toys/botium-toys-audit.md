# 🔐 Internal Security Audit — Botium Toys

**Portfolio Project | Google Cybersecurity Certificate**  
**Analyst:** Kevin | **Date:** June 2026  
**Framework:** NIST Cybersecurity Framework (CSF)

---

## 📋 Overview

This project documents a simulated internal IT security audit conducted for **Botium Toys**, a fictional small U.S. toy company with a growing online presence. The audit was performed as part of the Google Cybersecurity Certificate curriculum and is included here as a portfolio artifact demonstrating GRC and compliance analysis skills.

The audit follows the **NIST CSF Identify** function as its primary lens, producing a controls and compliance checklist with recommendations to improve the organization's overall security posture.

---

## 🏢 Company Profile

| Field | Detail |
|---|---|
| **Company** | Botium Toys (fictional) |
| **Location** | Single physical site — main office, storefront, warehouse |
| **Online Presence** | U.S. and international e-commerce |
| **Relevant Regulations** | PCI DSS (payment processing), GDPR (EU customers) |
| **Risk Score** | 8/10 (High) |

---

## 🎯 Audit Scope & Goals

**Scope:** The entire security program at Botium Toys — including employee equipment and devices, internal network, systems, and data practices.

**Goal:** Assess existing assets and complete a controls and compliance checklist to identify gaps and recommend controls/best practices needed to improve security posture.

---

## 🗂️ Asset Inventory

Assets managed by the IT Department:

- On-premises equipment for in-office business needs
- Employee end-user devices (desktops, laptops, smartphones, remote workstations, headsets, cables, keyboards, mice, docking stations, surveillance cameras)
- Storefront products available for retail sale on-site and online (stored in adjoining warehouse)
- Systems, software, and services: accounting, telecommunications, database, security, ecommerce, and inventory management
- Internet access and internal network infrastructure
- Data retention and storage
- Legacy systems requiring human monitoring (end-of-life)

---

## ⚠️ Risk Assessment Summary

| Field | Detail |
|---|---|
| **Risk Description** | Inadequate asset management; missing controls; potential non-compliance with U.S. and international regulations |
| **Primary NIST CSF Function** | Identify |
| **Risk Score** | 8 / 10 |
| **Impact Rating** | Medium (IT dept. unsure which assets would be at risk) |
| **Regulatory Risk** | High — missing controls and compliance practices for PCI DSS and GDPR |

### Key Risk Findings from Source Document

| Finding | Risk |
|---|---|
| All employees have access to internally stored data including cardholder data and PII/SPII | Confidentiality breach |
| No encryption on credit card data at rest or in transit | PCI DSS violation |
| No least privilege or separation of duties controls | Insider threat / privilege abuse |
| No IDS installed | Undetected intrusion |
| No disaster recovery plan or data backups | Business continuity failure |
| Password policy exists but doesn't meet complexity standards | Credential compromise |
| No centralized password management system | Operational risk |
| Legacy systems monitored without a formal schedule | Vulnerability exposure |
| Physical site has locks, CCTV, and fire detection ✅ | Adequate |
| Firewall with defined ruleset in place ✅ | Adequate |
| Antivirus installed and monitored regularly ✅ | Adequate |
| 72-hour EU breach notification plan in place ✅ | GDPR partial compliance |

---

## ✅ Controls Assessment Checklist

### Administrative / Managerial Controls

| Control | In Place? | Notes |
|---|---|---|
| Least Privilege | ❌ No | All employees can access all internal data |
| Disaster Recovery Plans | ❌ No | No DRP; no data backups |
| Password Policies | ⚠️ Partial | Policy exists but below complexity standards |
| Separation of Duties | ❌ No | Not implemented |
| Access Control Policies | ❌ No | No formal access control framework |
| Account Management Policies | ❌ No | No lifecycle management in place |

### Technical Controls

| Control | In Place? | Notes |
|---|---|---|
| Firewall | ✅ Yes | Defined ruleset actively in use |
| Intrusion Detection System (IDS) | ❌ No | Not installed |
| Backups | ❌ No | No backup of critical data |
| Antivirus (AV) Software | ✅ Yes | Installed and monitored regularly |
| Manual Monitoring for Legacy Systems | ⚠️ Partial | Monitored but no formal schedule or clear intervention methods |
| Encryption | ❌ No | Credit card and PII data not encrypted |
| Password Management System | ❌ No | No centralized system |

### Physical / Operational Controls

| Control | In Place? | Notes |
|---|---|---|
| Locks (offices, storefront, warehouse) | ✅ Yes | Adequate |
| CCTV Surveillance | ✅ Yes | Up-to-date; covers all locations |
| Fire Detection / Prevention | ✅ Yes | Fire alarms and sprinklers in place |
| Adequate Lighting | ✅ Yes | Assumed sufficient per physical site description |
| Locking Cabinets (network gear) | ❓ Unknown | Not explicitly confirmed |
| Time-Controlled Safe | ❓ Unknown | Not explicitly confirmed |
| Signage (alarm service provider) | ❓ Unknown | Not explicitly confirmed |

---

## 📜 Compliance Checklist

### PCI DSS — Payment Card Industry Data Security Standard

| Best Practice | Compliant? | Notes |
|---|---|---|
| Only authorized users have access to credit card data | ❌ No | All employees have access |
| Credit card data stored/processed in a secure environment | ❌ No | No encryption; stored in internal DB |
| Data encryption procedures implemented | ❌ No | Not currently in place |
| Secure password management policies adopted | ❌ No | Policy below standard; no enforcement system |

### GDPR — General Data Protection Regulation

| Best Practice | Compliant? | Notes |
|---|---|---|
| EU customers' data kept private/secured | ❌ No | No encryption; broad access |
| 72-hour breach notification plan for EU customers | ✅ Yes | Plan exists and is documented |
| Data properly classified and inventoried | ❌ No | Asset management is inadequate |
| Privacy policies enforced to document and maintain data | ✅ Partial | Policies exist among IT staff; enforcement unclear |

### SOC Type 1 / SOC Type 2

| Best Practice | Compliant? | Notes |
|---|---|---|
| User access policies established | ❌ No | No least privilege or access control policies |
| Sensitive data (PII/SPII) is confidential/private | ❌ No | Broad employee access; no encryption |
| Data integrity controls in place | ✅ Yes | IT dept. has integrated data integrity controls |
| Data available only to authorized individuals | ❌ No | No access restrictions in place |

---

## 📌 Recommendations

Based on the audit findings, the following controls should be prioritized for immediate implementation, ranked by risk impact:

### 🔴 Critical — Implement Immediately

1. **Encryption** — All credit card and PII/SPII data must be encrypted at rest and in transit to meet PCI DSS and GDPR requirements.
2. **Least Privilege + Access Control Policies** — Restrict employee access to only the data/systems needed for their role. Reduces insider threat and regulatory exposure significantly.
3. **IDS/IPS** — Install an intrusion detection system to identify anomalous traffic and potential breaches in real time.
4. **Disaster Recovery Plan + Backups** — Critical for business continuity. No backups means a ransomware attack or hardware failure could be catastrophic.

### 🟠 High Priority — Implement Within 90 Days

5. **Password Management System** — Deploy a centralized password manager to enforce complexity standards and reduce IT help desk load.
6. **Separation of Duties** — Formalize roles so no single employee can access and manipulate sensitive data without oversight.
7. **Legacy System Monitoring Schedule** — Formalize schedules and intervention protocols for end-of-life systems.

### 🟡 Medium Priority — Plan and Schedule

8. **Account Management Policies** — Define user lifecycle policies (onboarding, offboarding, role changes) to reduce attack surface from stale accounts.
9. **Data Classification and Inventory** — Support GDPR compliance and improve overall asset visibility.

---

## 🗃️ Files in This Repository

| File | Description |
|---|---|
| `README.md` | This document — full audit narrative and findings |
| `controls-compliance-checklist.md` | Standalone checklist in markdown format |
| `risk-assessment-summary.md` | Risk scoring and source findings summary |

---

## 🛠️ Skills Demonstrated

- Internal security audit methodology
- NIST CSF framework application (Identify function)
- Controls assessment (Administrative, Technical, Physical)
- Compliance evaluation (PCI DSS, GDPR, SOC 1/2)
- Risk scoring and prioritization
- Security posture gap analysis
- Technical documentation for stakeholder communication

---

## 📚 References

- NIST Cybersecurity Framework (CSF) — [nist.gov/cyberframework](https://www.nist.gov/cyberframework)
- PCI DSS — [pcisecuritystandards.org](https://www.pcisecuritystandards.org)
- GDPR — [gdpr.eu](https://gdpr.eu)
- Google Cybersecurity Certificate — Coursera
